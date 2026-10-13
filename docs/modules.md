# CoreHR — Module Reference

> **Portfolio documentation only.** This document summarises the implemented modules, their API surface, and their key behaviours as verified from the project codebase. No source code is reproduced.

---

## Module Map

| Module (backend package) | Frontend Pages / Routes | Purpose |
|---|---|---|
| `app/auth` | `/login`, `/forgot-password`, `/verify-otp`, `/reset-password`, `/change-password` | Authentication, sessions, 2FA, password reset |
| `app/employees` | `/dashboard/EmployeeManagement`, `/dashboard/employee` | Employee profile CRUD |
| `app/roles`, `app/departments`, `app/designations` | (admin-only UI panels) | RBAC, org structure |
| `app/recruitment` | `/dashboard/recruitment`, `/jobs/[id]` (public) | ATS, vacancies, applications, interview panels |
| `ai_service/` | (backend-only, no direct UI) | AI CV screening microservice |
| `app/leave` | `/apply-leave`, `/assign-leave`, `/approval`, `/leave`, `/leave-history`, `/leave-settings` | Leave requests, approvals, balances, accrual |
| `app/documents` | `/dashboard/documents`, `/documents` | Document upload, review, template-based generation, email requests |
| `app/time_tracking` | `/dashboard/time-tracking`, `/dashboard/attendance` | Clock-in/out, pause/resume, overtime |
| `app/messages` | (via dashboard messaging panel) | Internal messaging inbox/sent |
| `app/notifications` | `/dashboard/notifications` | In-app and email notification management |
| `app/announcements` | (via dashboard widget) | Company-wide announcements |
| `app/events` | `/dashboard/events` (via widget) | Company events management |
| `app/calendar_holidays` | (via dashboard widget) | Public holiday management |
| `app/dashboard` | `/dashboard` (main page) | Widget layout persistence |
| `app/reports` | `/reports`, `/dashboard/reports` | Leave and attendance reports |

---

## auth

**Backend:** `app/auth/router.py`, `app/auth/service.py`, `app/auth/models.py`

Key endpoints (summarised, not verbatim):

| Method | Path | Description |
|---|---|---|
| POST | `/auth/login` | Credential verification, 2FA check, token issuance |
| POST | `/auth/verify-2fa` | TOTP code verification for 2FA users |
| POST | `/auth/logout` | Refresh token revocation |
| POST | `/auth/refresh` | Issue new token pair from valid refresh token |
| GET | `/auth/me` | Current user profile and permissions |
| POST | `/auth/forgot-password` | Send OTP to email (same response regardless of account existence) |
| POST | `/auth/verify-otp` | Validate reset OTP |
| POST | `/auth/reset-password` | Set new password after OTP verification |
| POST | `/auth/first-login/password` | Mandatory first-login password change |
| GET | `/auth/security/2fa` | Generate TOTP QR code for 2FA setup |
| POST | `/auth/security/2fa` | Activate 2FA with TOTP code |
| DELETE | `/auth/security/2fa` | Disable 2FA (requires current password) |
| PUT | `/auth/security/password` | Change password (security settings) |

**Models:** `User`, `AuditLog`, `OTPRecord`

---

## employees

**Backend:** `app/employees/router.py`, `app/employees/service.py`

Key operations:
- Create, read, update, (soft-)delete employee records
- Upload and manage profile images
- View and update personal profile (self-service)
- Resend login details (HR/Admin only; revokes existing sessions)
- Designation history tracking

**Models:** `Employee`, `EmployeeDesignationHistory`

---

## roles / departments / designations

- Roles store a permission list (many-to-many via `role_permissions` join table)
- Users can hold multiple roles (many-to-many via `user_roles`)
- Departments and designations are hierarchical reference data used in employee records and leave reporting

---

## recruitment

**Backend:** `app/recruitment/router.py`, `app/recruitment/public_router.py`, `app/recruitment/service.py`

Key operations:
- Vacancy CRUD (Draft / Active / Closed lifecycle)
- Public job listing (unauthenticated) and application submission (with reCAPTCHA)
- CV upload and AI screening trigger
- Application status progression through pipeline
- Interview panel creation and member assignment
- Interview evaluation submission (5 criteria × 5 points)
- Final decision submission (panel head only)
- Automated candidate emails (scheduling, offer, rejection)

**Models:** `Vacancy`, `Application`, `InterviewPanel`, `InterviewEvaluation`, `FinalDecision`

**AI integration:** `app/core/ai_client.py` dispatches to `ai_service/` via httpx

---

## leave

**Backend:** `app/leave/router.py`, `app/leave/service.py`

Key operations:
- Leave type and entitlement management (admin)
- Employee self-service leave request
- HR-assigned leave on behalf of an employee
- Multi-tier approval (manager first, then HR/admin)
- Approval, rejection, resubmission, and cancellation
- Balance calculation (with per-role entitlement and proration)
- Medical leave conversion workflow
- Leave accrual rules per employee
- Reporting (filterable, with PDF export)

**Models:** `LeaveType`, `LeaveEntitlement`, `LeaveRequest`, `EmployeeLeaveAccrualRule`

---

## documents

**Backend:** `app/documents/routers/` (multiple routers)

Key operations:
- Employee document upload (any configured document type)
- HR review → approve or reject with reason
- Document type management (admin)
- Document template upload and management
- Promotion letter generation (from template + employee data)
- Document request portal (employee submits request for a letter/certificate)
- HR fulfils request → generates and attaches document
- IMAP email poller imports external requests (RFC 5322 Message-ID dedup)

**Models:** `EmployeeDocument`, `DocumentType`, `DocumentTemplate`, `DocumentRequest`, `DocumentAuditLog`

**File storage:** Routed through `app/core/storage_service.py` — local or S3/MinIO

---

## time_tracking

**Backend:** `app/time_tracking/router.py`

Key operations:
- Clock in (start session)
- Pause / Resume (creates `TimeCheckPair` records)
- Clock out / End work day (finalises `TimeEntry`)
- View personal time history (daily, weekly, monthly)
- HR/Admin view all employees' attendance
- Overtime threshold management (versioned, effective-date based)
- Auto-finalization of abandoned sessions at local midnight (APScheduler job)

**Models:** `TimeEntry`, `TimeCheckPair`, `OvertimeThresholdHistory`

---

## messages

**Backend:** `app/messages/router.py`

Key operations:
- Compose and send to target groups (All, All Employees, HR, Department)
- Inbox view (per recipient, with read/unread status)
- Sent view (per sender)
- Mark as read
- Soft-delete (sender and recipient independently) with permanent-delete option

**Models:** `Message`, `MessageRecipient`

---

## notifications

**Backend:** `app/notifications/router.py`, `app/notifications/service.py`

Key operations:
- Retrieve in-app notifications (paginated)
- Mark as read / mark all as read
- Update notification preferences (per category: email and in-app toggles)
- Update notification retention policy
- Internal push endpoint (service-to-service, key-protected)

**Models:** `Notification` (with `user_id`, `category`, `title`, `body`, `link`, `is_read`, `created_at`)

---

## announcements / events / calendar_holidays

- Announcements: CRUD with manage permission gate; broadcast to all active users
- Events: CRUD with manage permission gate; past dates blocked; notifications on create/update/cancel
- Holidays: date CRUD (2000–2100 range, duplicate prevention); used for holiday reminder notifications

---

## dashboard

- Single endpoint: GET / PUT `/dashboard/layout`
- Layout stored as JSONB in `dashboard_layouts` table, one row per user
- Widget sizes and positions validated server-side on PUT

---

## reports

- Leave reports: filterable by employee, leave type, date range; PDF export via ReportLab
- Time tracking reports: aggregate daily/weekly/monthly views

---

*This document is part of the CoreHR portfolio. It contains no confidential information about Insharp Technologies.*
