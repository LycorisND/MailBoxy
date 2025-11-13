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
  *** Begin Plan

  ````markdown
  # Implementation Plan: Telegram Mail Client

  **Branch**: `001-telegram-mail-client` | **Date**: 2025-11-14 | **Spec**: `specs/001-telegram-mail-client/spec.md`
  **Input**: Feature specification from `specs/001-telegram-mail-client/spec.md`

  ## Summary

  MailBoxy: enable users to link multiple mailbox providers via OAuth and
  receive unified mail in a Telegram-integrated experience. The feature
  provides: secure OAuth-based account linking, aggregated message listing in
  the web UI (opened from Telegram web-app), and Telegram notifications with
  OTP-first policy (full OTP placed in spoiler text with masked fallback).

  Initial technical approach: implement a web backend (FastAPI) that manages
  accounts, message aggregation and notification dispatch. A lightweight
  frontend (React/Vite SPA) will provide the web-app UI and be embedded in
  Telegram's web-app. A bot service will handle notifications and quick actions.

  ## Technical Context

  **Language/Version**: Python 3.11 (backend) + TypeScript/React (frontend)
  **Primary Dependencies**: FastAPI, SQLAlchemy, alembic, React + Vite, Redis + RQ/Celery (workers), HTTP client libs for provider APIs.
  **Storage**: PostgreSQL for metadata and message indices; optional S3-compatible object storage for full message bodies (persisted per-account opt-in).
  **Testing**: pytest for backend unit/integration; Playwright/Cypress for frontend E2E; contract tests for notification flows.
  **Target Platform**: Containerized Linux hosts behind HTTPS (production domain `mailboxy.app`).
  **Performance Goals**: Notification delivery within 30s for 95% deliveries; backend p95 API latency <200ms under expected load.
  **Constraints**: OAuth provider rate limits and consent surfaces; tokens encrypted at rest using KMS; avoid storing plaintext secrets in repo.

  ## Constitution Check

  This plan enforces the constitution gates from `spec.md` and the project's
  policy. The plan explicitly includes KMS integration, per-account retention
  opt-in, and OTP-first notification behavior.

  Gates (to be validated during Phase 0):
  - Privacy & Data Minimization: Data stored, scope, and retention are documented and enforced.
  - OAuth & Authentication: OAuth scopes and redirect URIs registered; re-auth for sensitive actions.
  - Notification Policy: OTP-first notifications delivered only to user's private chat; spoiler formatting with masked fallback.
  - Observability: Metrics for delivery latency and failure rates, and alerts for high failure rate.

  ## Implementation Plan (Phase breakdown)

  This plan converts the specification into actionable engineering phases,
  milestones and acceptance gates. Each phase produces artifacts in
  `specs/001-telegram-mail-client/` and maps to tasks in `tasks.md`.

  Phase 0 — Research & Design (2-4 days)
  - Deliverables: `research.md`, refined `data-model.md`, minimal OpenAPI contract and updated `spec.md` success criteria.
  - Actions:
    - Finalize OAuth scopes and provider-specific consent flows (T012, T019-T021).
    - Design encryption key lifecycle and select KMS provider (T051).
    - Define message persistence semantics and retention defaults (T049/T050).
    - Define metrics and alerts for observability (delivery latency, failure rate) (T017/T034).

  Phase 1 — Foundation & Scaffolding (3-5 days)
  - Deliverables: runnable backend/frontend/bot scaffold, DB migrations, CI workflows.
  - Actions:
    - Create project layout and dependency manifests (T001-T007).
    - Implement models and migrations for `User`, `MailboxAccount`, `Message`, `Notification` (T009, T013, T014).
    - Implement token encryption helper configured to fetch keys from KMS (T010, T051).
    - Setup basic observability (structured logs + metrics) and test harness (T017, T018).
    - Provision staging domain/tunnel and document OAuth redirect setup (T052, T054).

  Phase 2 — Core MVP (OAuth + Notifications) (5-10 days)
  - Deliverables: OAuth linking flow, message ingestion, OTP detection, Telegram notification dispatch.
  - Actions:
    - Implement OAuth start/callback endpoints and provider registry (T019-T022, T012).
    - Implement message ingestion pipeline: webhooks and/or pollers with backoff (T027).
    - Implement OTP detection heuristics and notification formatter with spoiler fallback (T028, T029).
    - Implement notification dispatcher + worker with retry/backoff and metrics (T030, T031, T034).
    - Add contract and integration tests for notification behavior (T032, T033).

  Phase 3 — Aggregation + UI (3-7 days)
  - Deliverables: Aggregated mail list API and Telegram web-app SPA.
  - Actions:
    - Persist message metadata and implement `GET /api/v1/messages` with pagination and filtering (T035, T036).
    - Implement frontend mail list and account management pages (T025, T037, T040).
    - Implement session handling with re-auth gate for sensitive operations (T042, T043).

  Phase 4 — Hardening & Release (2-4 days)
  - Deliverables: Security review, performance tests, deployment manifests, retention enforcement job.
  - Actions:
    - Add CI secret scanning and pre-merge checks (T053).
    - Implement retention enforcement worker to delete expired persisted bodies (T050).
    - Security review and dependency audit (T045).
    - Prepare k8s/Terraform manifests and run smoke tests (T048).

  Phase 5 — Post-launch Observability & Iteration (ongoing)
  - Deliverables: Dashboards, alerts, UX improvements.
  - Actions:
    - Monitor notification latency and error rates; tune retries/concurrency.
    - Improve OTP heuristics from real-world corpus and add opt-out controls.

  ## Acceptance Gates & Mapping
  - Gate A (Safety): KMS integration (T051) implemented and secrets not stored in plaintext.
  - Gate B (Functionality): OAuth linking and notification delivery tests pass (T019-T021, T031, T032).
  - Gate C (Privacy): Retention enforcement implemented and opt-in defaults applied (T049, T050).
  - Gate D (Observability): Metrics and alerts for delivery latency and failure rate in place (T017, T034).

  Each gate must be satisfied before the Feature is considered releasable. Tests for each gate must be automated and included in CI where practical.

  ## Estimates & Teaming
  - Rough estimate (MVP): 3-4 engineer-weeks (one engineer full-time), split as:
    - Research & foundation: 1 week
    - Core MVP (OAuth+Notifications): 1.5 weeks
    - Aggregation + UI: 0.5-1 week
    - Hardening & release: 0.5 week

  ## Risks & Mitigations
  - OAuth provider rate limits & consent UI changes — Mitigation: use provider-specific backoff and test accounts; keep scopes minimal.
  - Storing full message bodies — Mitigation: make message body persistence opt-in per account and enforce retention via scheduled job (T049/T050).
  - Secrets management — Mitigation: integrate KMS early and add CI scanning (T051, T053).

  ## Next Steps (immediate)
  1. Review this plan and confirm estimates & priority.
  2. Run Phase 0 research tasks (update `research.md`), then start Phase 1 scaffolding (T001..T017).
  3. After Phase 1, re-run constitution check and open a draft PR for the spec + plan.

  *** End Plan

  ``` 
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
