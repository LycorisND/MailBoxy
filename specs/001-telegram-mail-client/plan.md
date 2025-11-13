# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

MailBoxy: enable users to link multiple mailbox providers via OAuth and
receive unified mail in a Telegram-integrated experience. The feature
provides: secure OAuth-based account linking, aggregated message listing in
the web UI (opened from Telegram web-app), and Telegram notifications with
OTP-first policy (full OTP placed in spoiler text with masked fallback).

Initial technical approach: implement a web backend (REST) that manages
accounts, message aggregation and notification dispatch. A lightweight
frontend (SPA) will provide the web-app UI and be embedded in Telegram's
web-app. The Telegram bot will handle notifications and quick actions.

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: Python 3.11 (backend) + TypeScript/React (frontend) - REASON: fast API iteration and rich web-app UX.
**Primary Dependencies**: FastAPI (backend), SQLAlchemy or equivalent ORM, alembic (migrations), React + Vite (frontend), a simple task queue (Redis + RQ or Celery), HTTP client libs for provider APIs.
**Storage**: PostgreSQL for metadata and message indices; optional object storage (S3-compatible) for full message bodies if persisted.
**Testing**: pytest for backend unit/integration tests; Playwright or Cypress for frontend E2E; contract tests for notification flows.
**Target Platform**: Linux server (containerized) with HTTPS fronting (mailboxy.app).
**Project Type**: Web application (backend + frontend + bot service).
**Performance Goals**: Notification delivery within 30s for 95% messages; backend p95 API latency <200ms under normal load.
**Constraints**: OAuth providers rate limits; must avoid storing plain credentials; encryption keys kept out of repo.
**Scale/Scope**: Initial MVP target: hundreds to low thousands of users; design for horizontal scaling for notification workers.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*
## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Constitution gates (quick checklist for plan authors):

- **Privacy & Data Minimization**: Document what user data the feature
  accesses, stores, and retention periods. Indicate where sensitive data is
  encrypted at rest.
- **OAuth & Authentication**: If the feature integrates with external
  accounts, list required OAuth scopes, token lifecycle, and consent surface.
- **Notification Policy**: Describe how notifications will follow the
  project's OTP-first policy (which messages trigger high-priority alerts
  vs. silent delivery).
- **Observability**: List required metrics/logs/alerts for feature critical
  flows (auth, message processing, delivery failures).

Constitution gates assessment (referencing `spec.md`):

- Privacy & Data Minimization: PASS (spec requires minimal storage; tokens
  encrypted; retention documented in assumptions).
- OAuth & Authentication: PASS (spec mandates OAuth linking and re-auth for
  sensitive actions; plan will request scopes: mail.read-only, userinfo).
- Notification Policy: PASS (spec requires OTP-first policy and private
  chat delivery; OTP formatting specified as spoiler with fallback).
- Observability: PARTIAL — spec requires metrics/logs but plan will define
  specific metrics (delivery latency, failure rate) in Phase 0 research.

This section MUST be completed and linked to relevant spec items before a
plan can be approved.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```

**Structure Decision**: Web application with three services:

- `backend/` - FastAPI service: API endpoints, background workers, OAuth
  integration, notification dispatcher.
- `frontend/` - React SPA (Vite): web-app UI embedded in Telegram web-app.
- `bot/` - Telegram bot service (can live within backend as a subservice or
  a lightweight process) responsible for sending notifications and handling
  quick actions.

Tests grouped by service under `tests/` with `unit/`, `integration/`, and
`e2e/` for cross-service flows.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
