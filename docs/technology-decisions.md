# CoreHR — Technology Decisions

> **Portfolio documentation only.** This document explains the rationale behind key technology choices in the CoreHR project.

---

## Frontend: Next.js 16 with App Router

**Why Next.js over a pure React SPA?**

The App Router model allowed the team to mix server and client rendering at the component level. Pages that display mostly static HR data (employee lists, document templates) benefit from server-side rendering, while interactive sections (dashboard widgets, leave forms, the ATS Kanban) use client components with local state. This hybrid approach avoids the overhead of a full SSR framework for every page while still providing good initial load performance.

**Why TypeScript?**

The backend exposes structured JSON schemas through Pydantic models. TypeScript types were written to mirror those schemas, catching mismatches between API responses and frontend assumptions at compile time rather than at runtime. ESLint and tsc were run as part of the CI workflow.

---

## Backend: FastAPI

**Why FastAPI over Django REST Framework or Flask?**

FastAPI's async-first design aligns with the project's requirements: background tasks (email dispatch, LLM inference calls, file uploads) are offloaded as `BackgroundTasks` or via APScheduler, so the request handler returns immediately. Pydantic schemas provide automatic request/response validation and serialisation. The automatic OpenAPI documentation was useful during development for the team to explore and test endpoints.

**Why a modular monolith rather than microservices?**

The team evaluated a more decomposed architecture early on but decided against it. The operational complexity of running, deploying, and debugging multiple services was not justified by the scale of the project. Instead, each domain (leave, recruitment, documents, etc.) is a self-contained Python package with its own router, service, models, and schemas — the internal structure of a microservice, without the network boundary.

The one exception is the AI screening service, which was extracted specifically because LLM inference latency would harm the main API's responsiveness.

---

## Database: PostgreSQL 15

**Why PostgreSQL over MySQL or a NoSQL database?**

The project required relational integrity (foreign keys between employees, departments, designations, leave types), complex queries for reporting (leave balances aggregated by type and employee), and JSONB columns for semi-structured data (dashboard layouts, notification preferences). PostgreSQL's partial unique indexes were also essential for the soft-delete pattern.

**Alembic for migrations**

All schema changes are versioned through Alembic. This provides an auditable history of the schema and allows reversible migrations — important for the handover, where the receiving team needed to be able to roll back if a deployment went wrong.

---

## ORM: SQLAlchemy 2.0

SQLAlchemy 2.0's unified API (no more `legacy` vs `2.0 style`) and first-class support for async sessions were decisive factors. The 2.0-style `select()` queries are explicit and predictable, which helped reviewers understand what SQL was being generated without needing to trace through ORM magic.

---

## Authentication: JWT + bcrypt + pyotp

**JWT with HTTP-only cookies (not localStorage)**

Storing tokens in localStorage exposes them to XSS. HTTP-only cookies are not readable by JavaScript and are transmitted automatically on same-origin requests. The backend sets `Secure` and `SameSite=lax` flags in production.

**Short-lived access tokens**

Access tokens expire in 15 minutes. This limits the exposure window if a token is somehow leaked. Refresh tokens (7 days) are used to reissue access tokens transparently.

**bcrypt for password hashing**

bcrypt's adaptive cost factor makes brute-force attacks computationally expensive. The 72-byte effective input limit is documented and mirrored in the password policy to prevent silent truncation of long passwords.

**TOTP for 2FA**

Time-based OTP using pyotp is a widely understood standard (RFC 6238) compatible with any authenticator app. The TOTP secret is stored once at setup and is never returned by the API after that — the QR code is generated and shown once, then the secret is write-only.

---

## AI Screening: Multi-Provider, Configuration-Only Switching

**Why support multiple providers?**

AI API availability and cost change frequently. By abstracting the provider behind a single `AI_PROVIDER` / `AI_API_KEY` configuration, the team avoided coupling the system to a single vendor. The default provider in the environment template is Groq (free tier, high quota) for development convenience, with Gemini, OpenAI, and Claude as alternatives.

**Why a separate microservice?**

LLM inference can take 3–10 seconds. Running it inline in a FastAPI route handler would hold up the connection pool and make the API feel slow for other users. The microservice runs independently; the main backend dispatches a request and the result is stored when it returns.

---

## Background Jobs: APScheduler + PostgreSQL Advisory Locks

**Why not Celery + Redis?**

Celery requires a message broker (Redis or RabbitMQ) as additional infrastructure. For the scale of this project, APScheduler running inside the FastAPI process was simpler and sufficient. The distributed execution problem (multiple workers running the same job) was solved with PostgreSQL advisory locks — using the database that was already required, with no extra services.

**Advisory lock key derivation**

Lock keys are derived deterministically from the job name using SHA-256. This avoids accidental key collisions between jobs and avoids hard-coded magic numbers in the codebase.

---

## File Storage: Pluggable Backend (Local / S3)

**Why a storage abstraction?**

Development environments should not require AWS credentials. The `StorageService` abstraction lets developers use the local filesystem without changing any code. In production, setting `STORAGE_BACKEND=s3` and providing S3/MinIO credentials switches to object storage transparently. The `docker-compose.yml` includes MinIO for teams who want to test S3 behaviour locally.

---

## Document Generation: Jinja2 + xhtml2pdf + python-docx

HR-generated documents (promotion letters, service letters, salary confirmations) are produced from administrator-managed templates. The template engine (Jinja2) renders HTML or DOCX with placeholder substitution, and xhtml2pdf converts the result to a downloadable PDF. mammoth is used for DOCX-to-HTML conversion when needed in the review flow.

---

## Testing: pytest + pytest-asyncio

The backend test suite covers authentication flows, the first-login gate, session hardening, the OTP flow, temporary password expiry, and each module's validation schemas. Tests run with `cd backend && pytest -q`. The test suite was validated at the handover milestone: 134 tests passed, with 11 pre-existing failures in unrelated modules.

---

*This document is part of the CoreHR portfolio. It contains no confidential information about Insharp Technologies.*
