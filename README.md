# CoreHR — Enterprise Human Resource Management System

> **Portfolio Repository** — This repository documents and showcases the team's development work on the CoreHR HRMS project. It is not the original application repository and does not contain Insharp Technologies' application source code.

---

![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)
![Next.js](https://img.shields.io/badge/Next.js_16-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL_15-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS_v4-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)
![Python](https://img.shields.io/badge/Python_3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Repository Ownership and Handover](#2-repository-ownership-and-handover)
3. [Features and Modules](#3-features-and-modules)
4. [Technology Stack](#4-technology-stack)
5. [System Architecture](#5-system-architecture)
6. [Development Contributions](#6-development-contributions)
7. [Project Challenges and Technical Decisions](#7-project-challenges-and-technical-decisions)
8. [Portfolio Disclaimer](#8-portfolio-disclaimer)

---

## 1. Project Overview

**CoreHR** is a full-stack, enterprise-grade Human Resource Management System developed by a team of five developers for Insharp Technologies. The system was designed to digitise and automate the complete HR lifecycle — from recruiting and onboarding new employees through to daily operational workflows such as attendance, leave management, and internal communications.

### The Problem It Addresses

Insharp Technologies managed HR operations manually and across disconnected tools, resulting in:

- Time-consuming, error-prone leave request and approval processes
- No centralised record of employee documents and employment history
- Recruitment handled through ad-hoc email and spreadsheets with no pipeline visibility
- No standardised internal communications channel for HR announcements
- Inability to track employee attendance and working hours accurately

### The Solution

CoreHR provides a unified, role-based web platform where HR administrators, managers, and employees each have a tailored interface to carry out their responsibilities. The platform covers the full employee lifecycle — from the first job advertisement to day-to-day operational HR activities — and includes an AI-assisted candidate screening service to reduce manual CV review time.

### Project Status

The system was developed and delivered to Insharp Technologies as a completed product. The project has been handed over in full.

---

## 2. Repository Ownership and Handover

> The original source-code repository was transferred to Insharp Technologies at their request as part of the project handover. This repository serves as a portfolio documenting the project, its features, technologies, architecture, and development work. It is not the original application repository and does not contain Insharp Technologies' application source code.

This portfolio was created by team members to document the work performed during the engagement. It does not represent Insharp Technologies' current production system, nor does it imply any ongoing relationship with or endorsement by Insharp Technologies.

---

## 3. Features and Modules

### 3.1 Authentication and Security

The authentication system is built on JWT with short-lived access tokens (15 minutes) and longer-lived refresh tokens (7 days), stored as secure HTTP-only cookies. Key capabilities verified in the codebase:

| Capability | Status |
|---|---|
| JWT-based login with secure cookie storage |  Implemented |
| Time-based One-Time Password (TOTP) two-factor authentication (2FA) |  Implemented |
| First-login mandatory password change for new employees |  Implemented |
| Temporary password expiry (configurable, default 7 days) |  Implemented |
| Forgot password with OTP-based reset flow |  Implemented |
| Password policy enforcement (length, complexity, context-aware) |  Implemented |
| Rate limiting on login and OTP endpoints |  Implemented |
| Audit logging of authentication events |  Implemented |
| Session revocation on password change across all devices |  Implemented |

The password policy requires at minimum: 8–72 characters, uppercase and lowercase letters, a digit, a special character, and disallows passwords containing the user's name or email. The same policy is mirrored for live validation in the frontend.

---

### 3.2 Employee Management

Centralised employee records and lifecycle management, including:

| Capability | Status |
|---|---|
| Employee profile creation with full personal, contact, and bank details |  Implemented |
| Department and designation assignment |  Implemented |
| Designation history tracking (with start and end dates) |  Implemented |
| Soft-delete (deactivated employees retain historical records) |  Implemented |
| Partial-unique index enforcement (reuse of employee IDs and emails after deactivation) |  Implemented |
| Automated welcome email with temporary password on creation |  Implemented |
| "Resend login details" action for HR (with full session revocation) |  Implemented |
| Profile image upload |  Implemented |

Employee records are linked one-to-one with user accounts. Soft-deletion is used throughout so historical payroll, audit, and attendance data remains intact after an employee is deactivated.

---

### 3.3 Role-Based Access Control (RBAC)

The authorisation layer uses a granular, permission-based RBAC model. Permissions are stored in the database and assigned to roles; users can hold multiple roles.

| Permission Category | Examples |
|---|---|
| Widget visibility | `widget.time_tracking.view`, `widget.leave_balance.view`, `widget.calendar.edit` |
| Attendance | `attendance:view_others`, `time_tracking:edit_overtime_threshold` |
| Announcements and Events | `widget.announcements.manage`, `widget.upcoming_events.manage` |
| Messaging | `messaging.send` |
| Approvals | `widget.approval_summary.view_approvals`, `widget.approval_summary.view_requests` |
| Recruitment | `recruitment:manage`, `recruitment:interview_panel` (panel-scoped access) |
| Employee management | `employee:create` (for resend login details and employee creation) |

Predefined permission bundles exist for Employee, HR, and Super Admin roles. The Super Admin role has implicit access to all operations.

---

### 3.4 Recruitment Management (ATS)

A full Applicant Tracking System supporting multi-round hiring workflows:

| Capability | Status |
|---|---|
| Vacancy lifecycle: Draft → Active → Closed |  Implemented |
| Public job portal with candidate application form |  Implemented |
| Google reCAPTCHA v2 on public submission form |  Implemented |
| Duplicate application prevention (SHA-256 hash + unique email per vacancy) |  Implemented |
| CV upload (PDF and DOCX), with MIME-type and size validation |  Implemented |
| AI-assisted CV screening and scoring (see §3.5) |  Implemented |
| Kanban-style candidate pipeline (Uploaded → Called → First Round → Second Round → Offered/Rejected) |  Implemented |
| Dynamic interview panel creation and membership management |  Implemented |
| Panel-scoped access control (interviewers see only their assigned vacancies) |  Implemented |
| Multi-dimensional interview scoring (Technical, Communication, Cultural Fit, Leadership, Problem Solving — 5 criteria × 5 points) |  Implemented |
| Normalised aggregate scoring across panel members |  Implemented |
| Final decision workflow: Proceed / Job Offered / Rejected |  Implemented |
| Automated email notifications to candidates (scheduling, offer, rejection) |  Implemented |
| Audit trail for ATS state changes |  Implemented |

The candidate pipeline is tracked through a defined status vocabulary (`Uploaded`, `Called`, `First Round`, `Second Round Pending`, `Second Round`, `Job Offered`, `Rejected`), enforced on both the backend service layer and the frontend interface.

---

### 3.5 AI-Assisted CV Screening

An independent AI microservice handles CV parsing and candidate scoring so that LLM inference does not affect the latency of the main HR API.

| Capability | Status |
|---|---|
| Standalone FastAPI microservice on a separate port |  Implemented |
| CV text extraction from PDF and DOCX |  Implemented |
| Multi-provider AI support (Groq/Llama 3, Google Gemini, OpenAI GPT, Anthropic Claude) |  Implemented |
| Configurable provider and model via environment variables (no code changes needed) |  Implemented |
| 0–100 AI match score with professional reasoning text |  Implemented |
| Deterministic regex fallback for contact extraction if AI is unavailable |  Implemented |
| Primary and fallback model configuration for quota resilience |  Implemented |

The service accepts a CV file path and vacancy context (title, description, requirements, experience level) and returns the extracted candidate name, email, phone, a numeric match score, and a short reasoning statement. The main backend calls this service via HTTP and never imports from it directly, preserving the decoupled architecture.

---

### 3.6 Leave Management

A comprehensive leave management system with multi-tier approval:

| Capability | Status |
|---|---|
| Configurable leave types with optional entitlement limits |  Implemented |
| Per-role annual entitlement overrides |  Implemented |
| Employee self-service leave requests |  Implemented |
| Half-day leave requests |  Implemented |
| HR-assigned leave on behalf of an employee |  Implemented |
| Multi-tier approval workflow (Manager → HR/Admin) |  Implemented |
| Leave balance calculation with proration support |  Implemented |
| Medical leave conversion workflow |  Implemented |
| Leave resubmission after rejection |  Implemented |
| Leave history view for employees |  Implemented |
| Leave approval/rejection view for managers and HR |  Implemented |
| Leave accrual rules per employee |  Implemented |
| PDF leave report export |  Implemented |
| Email notifications on status changes |  Implemented |

The approval routing enforces separation of duties: an employee cannot approve their own leave, and HR-assigned leave is routed to a separate HR/Admin approver rather than the assigner.

---

### 3.7 Document Management

A document lifecycle system covering both employee-uploaded documents and HR-generated letters:

| Capability | Status |
|---|---|
| Employee document upload with configurable document types |  Implemented |
| Mandatory document tracking per employee profile |  Implemented |
| HR review and approval/rejection workflow with reason |  Implemented |
| Separation of Duties (SoD) routing (HR submissions escalate to Super Admin) |  Implemented |
| Document template management (HTML/DOCX → PDF generation) |  Implemented |
| Promotion letter generation from templates via Jinja2 |  Implemented |
| Employee document request portal (internal) |  Implemented |
| Inbound email document request parsing (IMAP poller) |  Implemented |
| RFC 5322 Message-ID idempotency for email-sourced requests (prevents duplicates) |  Implemented |
| Document request lifecycle: Pending → In Progress → Approved → Completed / Rejected |  Implemented |
| Flexible file storage: local filesystem or S3-compatible object store (MinIO/AWS S3) |  Implemented |
| UUID primary keys on document records to prevent enumerable download URLs |  Implemented |

---

### 3.8 Attendance and Time Tracking

A work-session tracking system with overtime calculation:

| Capability | Status |
|---|---|
| Clock-in / Clock-out per work session |  Implemented |
| Pause and Resume support (multiple check-in/out pairs per day) |  Implemented |
| Daily work summary (total hours, overtime) |  Implemented |
| Configurable overtime threshold (hours/day), versioned with effective dates |  Implemented |
| Historical overtime values frozen at the threshold that was active when finalized |  Implemented |
| UTC timestamp storage with local-timezone day boundary logic |  Implemented |
| Auto-finalization of abandoned sessions at local midnight |  Implemented |
| Auto-start of new day when an employee clocks in after midnight |  Implemented |
| Attendance view for the current employee |  Implemented |
| HR/Admin view of all employees' attendance |  Implemented |
| Week and period-based attendance report views |  Implemented |

---

### 3.9 Dashboard and Widgets

Each user has a personalised, draggable dashboard composed of role-filtered widgets. Widget layouts are persisted per user in the database.

| Widget | Description |
|---|---|
| Time Tracking | Current session timer and quick clock-in/out control |
| Leave Balance | Remaining days per leave type |
| Attendance | Weekly hours summary chart |
| Notifications | Recent unread system notifications |
| Calendar | Company holidays and events viewer/editor |
| Approval Summary | Pending approvals (managers/HR) or my pending requests (employees) |
| Announcements | Company-wide announcements |
| Upcoming Events | Scheduled company events |

Widget sizes and positions are validated server-side on save to prevent layout corruption.

---

### 3.10 Notifications and Messaging

**System Notifications**

| Capability | Status |
|---|---|
| In-app notifications for leave status changes, document approvals, events, holidays |  Implemented |
| Per-user notification preferences (in-app and email, per category) |  Implemented |
| Configurable notification retention (auto-purge after N days, or keep forever) |  Implemented |
| Holiday reminder notifications (background scheduler, 24-hour advance notice) |  Implemented |
| Distributed deduplication via database (prevents duplicate sends across worker processes) |  Implemented |

**Internal Messaging**

| Capability | Status |
|---|---|
| Compose and send messages to All Users, All Employees, HR, or specific Departments |  Implemented |
| Inbox and Sent views with read/unread tracking |  Implemented |
| Soft-delete with permanent-delete option |  Implemented |
| Permission-gated sending (`messaging.send`) |  Implemented |

---

### 3.11 Announcements, Events, and Calendar

| Capability | Status |
|---|---|
| Company-wide announcements (create, edit, delete) |  Implemented |
| Upcoming events (create, edit, cancel) |  Implemented |
| Event date stored in UTC; displayed in local time |  Implemented |
| Prevention of past-dated event creation |  Implemented |
| Event create/update/cancel notifications to all active users |  Implemented |
| Public holiday calendar management |  Implemented |
| Calendar view integrated into dashboard widget |  Implemented |

---

### 3.12 Reports

| Capability | Status |
|---|---|
| Leave report generation (filterable by employee, type, date range) |  Implemented |
| PDF export of leave reports via ReportLab |  Implemented |
| Time tracking / attendance reports (daily, weekly, monthly views) |  Implemented |

---

## 4. Technology Stack

### Frontend

| Technology | Version | Purpose |
|---|---|---|
| Next.js (App Router) | 16.1.1 | React framework with SSR and client components |
| React | 19.2.3 | UI component library |
| TypeScript | 5.x | Type-safe JavaScript |
| Tailwind CSS | 4.x | Utility-first styling |
| Framer Motion | 12.x | Animations and transitions |
| Recharts | 3.x | Charts for attendance and dashboard widgets |
| React Grid Layout | 1.x | Drag-and-drop dashboard widget grid |
| Tiptap | 3.x | Rich text editor (announcements, messages) |
| Lucide React | 0.577 | Icon library |
| React Hot Toast | 2.x | Toast notification UI |
| date-fns | 4.x | Date manipulation |
| react-google-recaptcha | 3.x | Google reCAPTCHA v2 on public forms |

### Backend — Main API

| Technology | Version | Purpose |
|---|---|---|
| FastAPI | 0.128.0 | Python async REST API framework |
| Python | 3.10+ | Language |
| SQLAlchemy | 2.0.45 | ORM (async-compatible) |
| Alembic | 1.18.4 | Database schema migrations |
| PostgreSQL | 15 | Relational database |
| psycopg2 | 2.9.11 | PostgreSQL adapter |
| APScheduler | 3.11.2 | Background job scheduling |
| python-jose | 3.5.0 | JWT encoding/decoding |
| passlib + bcrypt | 1.7.4 / 3.2.0 | Password hashing |
| pyotp | 2.9.0 | TOTP 2FA generation and validation |
| qrcode | 8.2 | 2FA setup QR code generation |
| fastapi-mail | 1.4.2 | SMTP email dispatch |
| python-docx + mammoth | 1.2.0 / 1.12.0 | DOCX document generation and parsing |
| xhtml2pdf + Jinja2 | 0.2.17 / 3.1.6 | HTML-to-PDF rendering from templates |
| ReportLab | 4.4.10 | PDF leave report generation |
| PyPDF2 | 3.0.1 | PDF text extraction |
| boto3 | 1.43.49 | AWS S3 / MinIO file storage client |
| httpx | 0.28.1 | Async HTTP client (backend to AI service) |
| pydantic-settings | 2.14.0 | Environment variable configuration |

### AI Microservice

| Technology | Purpose |
|---|---|
| FastAPI | REST API for the screening service |
| google-genai | Google Gemini LLM integration |
| openai | OpenAI GPT integration |
| anthropic | Anthropic Claude integration |
| Groq (openai SDK) | Groq / Llama 3 integration |
| PyPDF2 | PDF text extraction |
| python-docx | DOCX text extraction |

The AI service uses a unified configuration: setting `AI_PROVIDER` and `AI_API_KEY` in the environment is all that is needed to switch between Groq/Llama 3, Google Gemini, OpenAI, and Anthropic Claude — no code changes required.

### Infrastructure and Tooling

| Technology | Purpose |
|---|---|
| Docker Compose | Local development orchestration (PostgreSQL, Adminer, MinIO) |
| MinIO | S3-compatible local object storage for file uploads |
| Adminer | Database administration UI (development) |
| uvicorn | ASGI server for FastAPI services |
| PostgreSQL Advisory Locks | Distributed job coordination without Redis |
| pytest + pytest-asyncio | Backend test suite |

---

## 5. System Architecture

### High-Level Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            USER'S BROWSER                                │
│                 Next.js 16 — App Router (SSR + Client)                   │
│                   TypeScript · Tailwind CSS · Framer Motion              │
└─────────────────────────────────┬────────────────────────────────────────┘
                                  │ HTTPS / REST (port 3000 → 8000)
┌─────────────────────────────────▼────────────────────────────────────────┐
│                       MAIN BACKEND (FastAPI)                             │
│                                                                          │
│  ┌────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │    Auth    │  │  Employees  │  │ Recruitment │  │  Leave / Docs   │ │
│  │  (JWT/2FA) │  │  (RBAC)     │  │   (ATS)     │  │  Attendance     │ │
│  └────────────┘  └─────────────┘  └──────┬──────┘  └─────────────────┘ │
│                                          │ HTTP                          │
│  ┌────────────┐  ┌─────────────┐         │          ┌─────────────────┐ │
│  │ Messaging  │  │  Dashboard  │         │          │  APScheduler    │ │
│  │  Notifs    │  │  Widgets    │         │          │  (PG Adv. Lock) │ │
│  └────────────┘  └─────────────┘         │          └─────────────────┘ │
│                                          │                               │
│  SQLAlchemy ORM + Alembic Migrations     │      File Storage Abstraction │
└──────────────────────────┬───────────────┼────────────────┬──────────────┘
                           │               │                │
              ┌────────────▼──┐   ┌────────▼────────┐   ┌──▼──────────────┐
              │  PostgreSQL   │   │  AI Microservice │   │  Object Store   │
              │  (port 5432)  │   │  FastAPI + LLM  │   │  Local or MinIO │
              │               │   │  (port 8001)    │   │  (S3-compat.)   │
              └───────────────┘   └─────────────────┘   └─────────────────┘
```

### Architectural Decisions

**Modular Monolith Backend**
The main FastAPI application is structured as a modular monolith. Each functional area (authentication, employees, recruitment, leave, documents, etc.) is a self-contained Python package with its own router, service, models, and schemas. This keeps the codebase navigable while avoiding the operational overhead of a full microservices deployment.

**Decoupled AI Microservice**
The AI screening service runs as a separate FastAPI application (port 8001). The main backend calls it via async HTTP using `httpx`. This separation ensures that slow LLM inference calls do not block or degrade the main HR API.

**Distributed Scheduling Without Redis**
Background jobs (holiday reminders, inbound email polling, session maintenance) use APScheduler with PostgreSQL advisory locks (`pg_try_advisory_lock`). In a multi-worker deployment, every worker races for the lock; only one wins and executes the job. This eliminates the need for Redis or Celery while being safe for any number of workers.

**Pluggable File Storage**
The `StorageService` abstraction supports both local filesystem and any S3-compatible object store (MinIO or AWS S3) through a single `STORAGE_BACKEND` environment variable. Switching from development local storage to production S3 requires no code changes.

**Soft Deletes for Audit Integrity**
Both `users` and `employees` tables use a `SoftDeleteMixin`. Records are flagged `is_deleted = true` rather than physically removed. Partial unique indexes scoped to `WHERE is_deleted = false` enforce uniqueness among active records while allowing re-use of identifiers after deactivation.

**Security Layering on Public Endpoints**
The public job portal applies multiple validation layers before touching the database: payload size limits, MIME-type verification for CV uploads, Google reCAPTCHA v2 verification, and a cryptographic SHA-256 duplicate-detection hash on the CV file content.

For more detail, see [docs/architecture.md](docs/architecture.md).

---

## 6. Development Contributions

The project was built by a team of five developers. The following is based on verified Git commit history.

### Team Members (from Git history)

| Contributor | Git Author Name(s) |
|---|---|
| Thisaru Ramanayake | `Thisaru Ramanayaka` |
| Sachintha Dilshan | `sachintha dilshan` |
| Sanduni Liyanage | `SanduniLiyanage`, `Sanduni Liyanage` |
| Nagitha Dilshan | `NagithaDS` |
| Prageeth Madusanka | `prageeth20010311-del`, `Prageeth Madusanka` |

> **Approximate commit counts from `git log`:**
> Sachintha Dilshan ~112 · Thisaru Ramanayake ~60 · Sanduni Liyanage ~48 · Nagitha DS ~20 · Prageeth Madusanka ~16
>
>  Commit count is not an accurate measure of contribution. Large features may land in a single commit; refactors may produce many small ones.

### Contribution Areas (from commit messages and code authorship)

| Module / Area | Key Contributors (from Git history) |
|---|---|
| Recruitment ATS, Kanban pipeline, interview rounds | Thisaru Ramanayake, Sachintha Dilshan |
| AI screening microservice and multi-provider integration | Thisaru Ramanayake |
| Document management (upload, templates, PDF generation, IMAP poller) | Sachintha Dilshan |
| Leave management (requests, approvals, accrual, medical conversion, reports) | Nagitha DS, Prageeth Madusanka |
| Authentication, security hardening, first-login flow, password policy | Sanduni Liyanage |
| Time tracking and attendance, timezone handling | Sanduni Liyanage, Sachintha Dilshan |
| Dashboard widgets, notifications, messaging, announcements, events | Sanduni Liyanage, Sachintha Dilshan |
| Employee management, RBAC, departments, sidebar UI | Thisaru Ramanayake, Sachintha Dilshan |
| Database migrations, seed scripts, test suite | All contributors |
| Final handover hardening and deployment documentation | Sanduni Liyanage |

>  These attributions are derived from Git commit metadata. The exact scope of each person's contribution to any given module should be verified directly with the team. Collaborative branches, pair programming, and code reviews may not be fully reflected in commit authorship alone.

---

## 7. Project Challenges and Technical Decisions

### Challenge 1: Distributed Background Jobs Without Additional Infrastructure

**Problem:** APScheduler runs independently in each worker process. With multiple uvicorn workers, scheduled jobs (holiday reminders, email polling) would fire N times per cycle — sending duplicate notifications.

**Decision:** PostgreSQL advisory locks (`pg_try_advisory_lock`) were used as a lightweight distributed mutex. At fire time, all workers race; exactly one obtains the lock, executes the job, and releases it. If the winner crashes, PostgreSQL automatically releases the lock on session close — no stale locks. This eliminated the need for Redis or Celery, using infrastructure that was already running.

---

### Challenge 2: Decoupling AI Inference from the Main API

**Problem:** LLM inference for CV screening can take several seconds. Running it inline would block request handlers and degrade API responsiveness for other users.

**Decision:** The AI screening logic was extracted into a standalone FastAPI microservice running on a separate port. The main backend communicates with it via async HTTP (`httpx`). A configuration-only approach to provider switching (Groq, Gemini, OpenAI, Claude) was implemented so the system can adapt to API availability and cost without code changes.

---

### Challenge 3: Reliable Inbound Email Request Deduplication

**Problem:** The original inbound email poller used the IMAP `\Seen` flag as the "already processed" marker. This is shared mutable state — any mail client opening the inbox silently clears it, causing the poller to re-import requests.

**Decision:** The `source_message_id` column (storing the RFC 5322 `Message-ID` header) was introduced as the authoritative deduplication ledger. A partial unique index in the database enforces uniqueness at the storage layer — so even two concurrent pollers cannot produce duplicate records from the same email.

---

### Challenge 4: Unique Constraints After Soft Deletion

**Problem:** Standard unique indexes on `email` and `employee_id` prevent reuse of those values after a soft-delete, which is a real operational need (e.g. a former employee rejoins).

**Decision:** Partial unique indexes were used: `WHERE is_deleted = false`. Only active records are checked for uniqueness. Soft-deleted records are excluded from the constraint.

---

### Challenge 5: Separation of Duties in Document Approvals

**Problem:** In a small HR team, the same person who submits a document request might also be the default approver — undermining internal controls.

**Decision:** A Separation of Duties routing engine was implemented: submissions made by HR-role users are automatically escalated to the Super Admin level rather than being self-approved.

---

### Challenge 6: Timezone-Correct Attendance Day Boundaries

**Problem:** Work sessions stored in UTC should roll over to a new "day" at local midnight, not UTC midnight — a mismatch causes sessions to be attributed to the wrong date.

**Decision:** All timestamps are stored in UTC. The `APP_TZ_OFFSET_MINUTES` environment variable (set to 330 for Sri Lanka, UTC+5:30) is used to compute local midnight when determining day boundaries for session roll-over, auto-finalization, and overtime calculation.

---

## 8. Portfolio Disclaimer

This repository is maintained by former development team members for portfolio and documentation purposes only.

- **It is not the original application repository.** The original repository was transferred to Insharp Technologies as part of the project handover.
- **It does not provide access to Insharp Technologies' production system,** database, or any live data.
- **It does not contain the application source code.** Only documentation, architecture descriptions, and high-level feature summaries are included.
- **This portfolio was created following the completed handover of the project to Insharp Technologies.** Publication approval for this portfolio has not been explicitly obtained from Insharp Technologies. If you intend to make this repository public, you should confirm with them that sharing this high-level documentation is acceptable.
- **No confidential information is included.** Company names, private infrastructure details, credentials, production URLs, customer data, and employee records have been deliberately excluded.

---

## Documentation

| Document | Description |
|---|---|
| [Architecture Deep Dive](docs/architecture.md) | Expanded architecture documentation with module relationships |
| [Technology Decisions](docs/technology-decisions.md) | Rationale for key technology choices |
| [Module Reference](docs/modules.md) | Detailed module-by-module feature reference |

---

*Portfolio created: October 2026. Last updated: October 2026.*
