# CoreHR — Architecture Deep Dive

> **Portfolio documentation only.** This document contains no credentials, internal URLs, private infrastructure details, or confidential company information.

---

## Overview

CoreHR is built as a **modular monolith** backend paired with a **decoupled AI microservice**. The decision to use a modular monolith rather than a full microservices architecture was deliberate: it reduces operational complexity while still preserving clear internal module boundaries that make the code maintainable and testable.

---

## Component Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         FRONTEND LAYER                          │
│                                                                 │
│  Next.js 16 (App Router)                                        │
│  ├── Server Components (data-fetching, layouts)                 │
│  ├── Client Components (interactive widgets, forms, modals)     │
│  ├── Context Providers (auth state, dialog state)               │
│  └── Custom Hooks (API wrappers, auth guards)                   │
└──────────────────────────────┬──────────────────────────────────┘
                               │ REST API over HTTPS
                               │ Secure HTTP-only cookie auth
┌──────────────────────────────▼──────────────────────────────────┐
│                     MAIN BACKEND (FastAPI)                      │
│                                                                 │
│  Modules (each: router + service + models + schemas)            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  auth        │  │  employees   │  │  recruitment         │  │
│  │  (JWT, 2FA,  │  │  (profiles,  │  │  (vacancies, ATS,    │  │
│  │   TOTP, OTP) │  │   softdelete)│  │   panels, scoring)   │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  leave       │  │  documents   │  │  time_tracking       │  │
│  │  (requests,  │  │  (upload,    │  │  (clock-in/out,      │  │
│  │   approvals, │  │   templates, │  │   overtime, pause/   │  │
│  │   accrual)   │  │   IMAP poll) │  │   resume, reports)   │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  messages    │  │  notifications│  │  dashboard           │  │
│  │  (inbox,     │  │  (in-app,    │  │  (widget layout,     │  │
│  │   sent, soft │  │   email,     │  │   JSONB persistence) │  │
│  │   delete)    │  │   prefs)     │  │                      │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐                             │
│  │  roles /     │  │  announcements│                            │
│  │  departments │  │  events       │                            │
│  │  designations│  │  calendar     │                            │
│  └──────────────┘  └──────────────┘                             │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Core Layer                                             │   │
│  │  config · deps · jwt · security · email · storage_svc  │   │
│  │  password_policy · rate_limit · scheduler · ai_client  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  SQLAlchemy 2.0 ORM + Alembic Migrations                        │
└──────────┬─────────────────────────────────────┬───────────────┘
           │                                     │ httpx (async)
  ┌────────▼─────┐                    ┌──────────▼──────────────┐
  │  PostgreSQL  │                    │  AI Screening Service   │
  │  (primary    │                    │  FastAPI + screener.py  │
  │   data store)│                    │  ─ Groq / Llama 3       │
  │              │                    │  ─ Google Gemini        │
  │  Advisory    │                    │  ─ OpenAI GPT           │
  │  Locks for   │                    │  ─ Anthropic Claude     │
  │  scheduler   │                    └─────────────────────────┘
  └──────────────┘
           │
  ┌────────▼─────────────┐
  │  File Storage        │
  │  StorageService      │
  │  (abstract layer)    │
  │  ─ local filesystem  │
  │  ─ MinIO / AWS S3    │
  └──────────────────────┘
```

---

## Database Schema Overview

The database is managed by Alembic. Key tables and their relationships:

```
users ─────────────────────────────────── one-to-one ──── employees
  │                                                           │
  ├── user_roles (many-to-many) ──── roles ──── role_permissions ──── permissions
  │
  ├── refresh_token (column)
  ├── must_change_password (bool)
  ├── temp_password_expires_at (nullable timestamp)
  ├── notification_preferences (JSONB)
  └── two_factor_enabled / totp_secret

employees
  ├── department_id ──── departments
  ├── designation_id ──── designations
  └── employee_designation_history (one-to-many)

vacancies
  ├── applications (one-to-many, cascade delete)
  │     ├── interview_evaluations (one-to-many)
  │     └── final_decisions (one-to-many)
  └── interview_panels (one-to-one, cascade delete)

leave_types
  └── leave_entitlements (per-role overrides, many-to-many via role_id)

leave_requests
  └── leave_type_id ──── leave_types

employee_documents (UUID PK)
document_requests (UUID PK, partial unique on source_message_id)
document_templates
document_types
document_audit_logs

time_entries (one per calendar day per user)
  └── time_check_pairs (multiple per day — pause/resume segments)

overtime_threshold_history (versioned, effective_date-based)

messages
  └── message_recipients (one-to-many)

notifications
dashboard_layouts (JSONB widget configuration per user)
announcements
events
holidays
```

---

## Authentication Flow

```
1. POST /auth/login
   ├── Rate-limit check (per IP)
   ├── Password verify (bcrypt)
   ├── Check must_change_password → 403 + first-login redirect if true
   ├── Check temp_password_expires_at → 401 if expired
   ├── If 2FA enabled → return { require_2fa: true }
   │     └── POST /auth/verify-2fa → issue tokens
   └── Issue access token (15 min) + refresh token (7 days) as HTTP-only cookies

2. Token Refresh
   └── POST /auth/refresh → validate refresh token, issue new pair

3. Protected routes
   └── get_current_user (core/deps.py)
         ├── Decode JWT, check jti, check is_active
         ├── If must_change_password → allow only allowlisted paths
         └── Attach user + permissions to request context
```

---

## Recruitment Pipeline

```
Vacancy: Draft → Active → Closed

Application status flow:
  Uploaded
    └─► Called
          └─► First Round
                └─► Second Round Pending
                      └─► Second Round
                            ├─► Job Offered
                            └─► Rejected

On upload:
  1. CV file validated (MIME type, size, SHA-256 dedup)
  2. Saved to storage (local or S3)
  3. AI screening triggered (async HTTP → ai_service)
  4. ai_score + ai_reasoning stored on application

Interview scoring:
  5 dimensions × 5 points = 25 max per evaluator
  Aggregate = sum(all evaluators' totals) / (25 × evaluator count) × 100

Final decision:
  Panel head submits → status moves to Job Offered / Rejected / Second Round
  Automated email sent to candidate
```

---

## Document Request Flow

```
Internal (portal):
  Employee → POST /document-requests → PENDING
  HR views → marks IN_PROGRESS → generates document → COMPLETED
  OR HR rejects → REJECTED

External (email):
  IMAP Poller (APScheduler, every 5 min) reads inbox
  ├── Extracts Message-ID (RFC 5322)
  ├── Checks document_requests.source_message_id (partial unique index)
  ├── If already imported → skip (idempotent)
  └── If new → creates DocumentRequest with source='EXTERNAL'
```

---

## Scheduler Architecture

```
APScheduler (AsyncIOScheduler)
  ├── holiday_reminder_loop (every 1 hour)
  │     └── pg_try_advisory_lock("holiday_reminder")
  │           └── only one worker per cycle sends notifications
  │
  ├── poll_email_inbox (every EMAIL_POLL_INTERVAL_SECONDS, default 300)
  │     └── pg_try_advisory_lock("poll_email_inbox")
  │
  ├── auto_end_abandoned_sessions (daily at local midnight)
  │     └── finalizes TimeEntry rows still "active" at end-of-day
  │
  └── ... (other maintenance jobs)

Advisory lock key derivation:
  SHA-256(job_name)[:15 hex chars] mod 2^62
  → stable bigint, fits PostgreSQL signed 64-bit advisory lock key
```

---

## File Storage Abstraction

```python
# Switch via environment variable — no code changes needed
STORAGE_BACKEND=local   # → LocalStorageBackend
STORAGE_BACKEND=s3      # → S3StorageBackend (MinIO or AWS S3)

# Unified API
storage.upload(data: bytes, key: str) → str
storage.get_url(key: str) → str
storage.download(key: str) → bytes
storage.file_exists(key: str) → bool
storage.delete(key: str) → None
storage.serve(key: str, filename: str) → Response
```

---

## Security Measures Summary

| Area | Mechanism |
|---|---|
| Authentication tokens | JWT, HTTP-only cookies, 15-min access / 7-day refresh |
| Token uniqueness | `jti` (JWT ID) claim — unique per token |
| Password storage | bcrypt (passlib), 72-byte effective limit |
| 2FA | TOTP (pyotp), QR code setup, password required to disable |
| Rate limiting | In-memory per-IP limiter (configurable; note: per-worker in multi-worker deploy) |
| First login | Temporary password flow with expiry; API gate blocks all other endpoints |
| Session revocation | Refresh token invalidated on password change; access tokens expire within 15 min |
| Soft delete | `is_deleted` flag + partial unique indexes — deactivated users cannot log in |
| Public form protection | reCAPTCHA v2, MIME validation, SHA-256 dedup, payload size limits |
| SoD (documents) | HR submissions auto-escalated to Super Admin |
| Download URL security | UUID primary keys on document records (not sequential integers) |
| CORS | Configured per-environment via `CORS_ORIGINS` |
| Proxy headers | `TRUST_PROXY_HEADERS` — must be set explicitly to prevent IP spoofing |

---

*This document is part of the CoreHR portfolio. It contains no confidential information about Insharp Technologies.*
