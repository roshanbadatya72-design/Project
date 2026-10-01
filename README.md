# TerraLedger ESG workspace

A React and FastAPI workspace for enterprise ESG data collection, approvals, emissions calculations, and BRSR working drafts. The frontend opens on a sign-in page. Sign-in requests a bearer token from the API; the separate demo-entry option opens the dashboard with local sample data.

> BRSR output is a working draft only. Validate reporting boundaries, factor sources, applicable SEBI requirements, and every disclosure with qualified internal reviewers before use. Nothing generated here is certified or filing-ready.

## What is included

- Executive sustainability dashboard with hierarchy and reporting-period controls, site coverage, approval queue, and emissions trend.
- Site-operator three-step submission flow; manager approval and return workflow; reporting and emissions views.
- FastAPI endpoints for JWT authentication, organization hierarchy, scoped submissions, approval decisions, emissions roll-ups, and HTML BRSR working drafts.
- SQLAlchemy data model and MySQL 8 schema for hierarchy nodes, assignments, metrics, dated emission factors, submissions, and audit events.
- Seed command for six subsidiaries, six business units, 250 India project sites, and an environment-configured group administrator.

## Run the frontend

Requires Node.js 20.19+ or 22.12+.

```powershell
npm.cmd install
npm.cmd run dev
```

Open the Vite URL printed in the terminal (normally `http://localhost:5173`). Build the production frontend with `npm.cmd run build`.

## Run MySQL and the API

Requires Docker Desktop with Compose. Copy `.env.example` to `.env`, then replace all sample passwords and the JWT secret with unique random values. Keep `.env` private. Set `BOOTSTRAP_EMAIL`, `BOOTSTRAP_NAME`, and a `BOOTSTRAP_PASSWORD` of at least 14 characters for the first group ESG lead.

```powershell
docker compose up --build -d
docker compose exec api python -m app.seed
```

The API is at `http://localhost:8000`; interactive API documentation is at `http://localhost:8000/docs`. The Compose configuration enables SQLAlchemy schema creation for local development. For controlled environments, set `AUTO_CREATE_SCHEMA=false` and apply the reviewed DDL in `backend/schema.sql` through your database migration process.

The seed command is idempotent for the initial organization, user, role assignment, and factor records. It refuses to run without bootstrap credentials. Do not use sample factor values for external reporting without confirming the applicable source, version, unit, and reporting period.

### Local Python alternative

Create a virtual environment, install `backend/requirements.txt`, configure the variables in `.env`, and start MySQL separately. From the `backend` directory, run `python -m app.seed` and then `uvicorn app.main:app --reload`. The `.env` file must be available to the process. This alternative uses the same MySQL database URL.

## API outline

- `POST /api/auth/token`: OAuth2 password flow; returns a short-lived bearer token.
- `GET /api/auth/me`: current identity and assigned roles.
- `GET /api/org/nodes`: only assigned nodes and descendants.
- `GET /api/submissions?status=submitted`: only submissions within the caller's assigned hierarchy.
- `POST /api/submissions`: site-level data entry; requires an entry role on the site or an ancestor assignment.
- `POST /api/submissions/{id}/decision`: approve or return a submitted record; requires an authorized manager assignment for that scope.
- `GET /api/analytics/rollup?node_id=1&period_start=2024-04-01&period_end=2025-03-31`: approved activity data and quantified Scope 1/2 totals for authorized descendants.
- `GET /api/brsr/draft?...`: HTML working draft for an authorized group ESG lead or auditor.

Every hierarchy query is scoped from persisted role assignments; client-supplied node IDs do not grant access. Scope 3 is not included in the quantified total, and missing factor matches remain unquantified rather than being treated as zero. Configure production secrets, TLS, backups, retention, migrations, and an organization-approved factor library before deployment.

## Roles

`group_esg_lead`, `subsidiary_manager`, `business_unit_manager`, `site_operator`, and `auditor` are stored as assignments to hierarchy nodes. Assignments inherit visibility down their node's descendants. Entry is restricted to project-site nodes; approvals require manager roles. BRSR generation requires a group ESG lead or auditor role at the requested scope.

## Current boundaries

The login form calls the API's OAuth2 token endpoint, but the dashboard still uses local sample values and its form actions do not yet call the authenticated API. Production use needs dashboard API integration, database migrations and backup policy, verified emission factors, and organization-specific BRSR validation. No real corporate or site data is bundled.
