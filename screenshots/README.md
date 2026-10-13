# Screenshots

This directory is reserved for application screenshots that are suitable for inclusion in the portfolio.

## Guidelines Before Adding Screenshots

Before adding any screenshot to this directory, verify that it does **not** contain:

- Real employee names, employee IDs, or personal information
- Real email addresses, phone numbers, or contact details
- Candidate CVs, application data, or personal information from job applicants
- Real financial information (salary, bank accounts)
- Company-identifying information that has not been approved for publication
- Credentials, API keys, tokens, or environment configuration values
- Any information about the client company's internal systems or infrastructure

## Recommended Content

Suitable screenshots would include:

- The login page (no filled credentials)
- The dashboard with dummy/anonymised widget data
- The recruitment pipeline Kanban view with sample (non-real) candidate names
- The leave request form (empty or with example dates)
- The document management interface with example document types
- The time tracking widget (anonymised user)

## Status

⚠️ **No screenshots have been added yet.**

To add screenshots: take them from a local development environment with seeded test data (not production), verify they contain no personal or confidential information, and place them in this directory. Then update the main `README.md` to reference them in a Screenshots section.

The backend provides seed scripts (`backend/seed.py`, `backend/seed_test_users.py`) that populate the database with realistic but entirely synthetic test data, which is suitable for generating portfolio screenshots.
