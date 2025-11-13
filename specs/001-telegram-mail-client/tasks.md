---
description: "Task list for Telegram Mail Client feature"
---

# Tasks: Telegram Mail Client

**Input**: Design documents from `specs/001-telegram-mail-client/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`

## Format: `[ID] [P?] [Story] Description`

## Phase 1: Setup (Project initialization)

- [ ] T001 Initialize repository layout for feature (backend/frontend/bot) - create `backend/`, `frontend/`, `bot/`, `tests/` directories
- [ ] T002 [P] Create `backend/` Python project and `pyproject.toml` (Python 3.11) — path: `backend/`
- [ ] T003 [P] Create `frontend/` React + Vite project — path: `frontend/`
- [ ] T004 [P] Create `bot/` service scaffold (Python or Node) with basic run script — path: `bot/`
- [ ] T005 Add CI workflow for linting, tests and safety checks — file: `.github/workflows/ci.yml`
- [ ] T006 Add `requirements.txt` and pin initial dependencies for backend — file: `backend/requirements.txt`
- [ ] T007 Add `package.json` and scripts for frontend — file: `frontend/package.json`
- [ ] T008 Create local dev quickstart docs (copy `quickstart.md` to `specs/001-telegram-mail-client/quickstart.md` if not present) — file: `specs/001-telegram-mail-client/quickstart.md`

## Phase 2: Foundational (Blocking prerequisites)

- [ ] T009 Setup PostgreSQL schema and migration tooling (alembic) — files: `backend/alembic/`, `backend/alembic.ini`
- [ ] T010 [P] Implement encrypted token storage helper and config for encryption key retrieval — file: `backend/src/lib/encryption.py`
- [ ] T011 Configure environment and secrets handling for local/dev and prod (env var templates) — file: `backend/.env.example`
- [ ] T012 Implement OAuth client config handling and provider registry (Gmail, Outlook, Yandex) — file: `backend/src/services/oauth_registry.py`
- [ ] T013 Implement database models for `User`, `MailboxAccount`, `Message`, `Notification` using SQLAlchemy — file: `backend/src/models/*.py`
- [ ] T014 Create initial migration for models (alembic revision) — files: `backend/alembic/versions/*.py`
- [ ] T015 [P] Implement background worker scaffold (RQ/Celery) for message sync and notification dispatch — file: `backend/src/workers/worker.py`
- [ ] T016 Setup Telegram bot configuration and basic send message helper using `TELEGRAM_BOT_TOKEN` — file: `bot/src/telegram_client.py`
- [ ] T017 Add structured logging and basic metrics scaffolding (prometheus client hooks) — file: `backend/src/lib/observability.py`
- [ ] T018 Add basic end-to-end test harness setup (pytest + test DB config) — file: `tests/e2e/conftest.py`

## Phase 3: User Story 1 - Connect / Remove Mailboxes (Priority: P1) 🎯 MVP

**Goal**: Allow users to connect and remove mailbox accounts via OAuth.
**Independent Test**: A test account completes OAuth and account appears in list.

- [ ] T019 [US1] [P] Implement `POST /api/v1/accounts` (initiate OAuth) — file: `backend/src/api/accounts.py`
- [ ] T020 [US1] Implement OAuth callback endpoint to finalize linking and persist encrypted tokens — file: `backend/src/api/accounts.py`
- [ ] T021 [US1] Implement `DELETE /api/v1/accounts/{accountId}` to remove/revoke tokens — file: `backend/src/api/accounts.py`
- [ ] T022 [US1] Create `MailboxAccount` model persistence logic and token encryption integration — file: `backend/src/models/mailbox_account.py`
- [ ] T023 [US1] Add unit tests for OAuth flow (mock provider) — file: `tests/unit/test_oauth_flow.py`
- [ ] T024 [US1] Add integration test: complete OAuth end-to-end with test provider config — file: `tests/integration/test_accounts_integration.py`
- [ ] T025 [US1] Add frontend OAuth "Add account" button and flow starter; show account in account list — file: `frontend/src/pages/Accounts.jsx`
- [ ] T026 [US1] Add backend validation and error handling for denied consent — file: `backend/src/api/accounts.py`

## Phase 4: User Story 2 - Notifications in Telegram (Priority: P1)

**Goal**: Detect incoming mail and deliver Telegram notifications (OTP-first).
**Independent Test**: Send test emails with and without OTP and verify bot messages.

- [ ] T027 [US2] Implement message ingestion pipeline for incoming mail events (webhook or poll handler) — file: `backend/src/services/message_ingest.py`
- [ ] T028 [US2] Implement OTP detection heuristics module (regex-based) and config — file: `backend/src/lib/otp_detection.py`
- [ ] T029 [US2] Implement notification formatter including Telegram spoiler markup and masked fallback — file: `backend/src/services/notification_formatter.py`
- [ ] T030 [US2] Implement notification dispatcher that queues jobs to worker — file: `backend/src/services/notification_dispatcher.py`
- [ ] T031 [US2] Implement worker task to call Telegram client with retry/backoff — file: `backend/src/workers/notifications.py`
- [ ] T032 [US2] Add contract test for `/api/v1/notifications` webhook (queueing behavior) — file: `tests/contract/test_notifications_contract.py`
- [ ] T033 [US2] Add integration tests to verify OTP vs non-OTP notification behavior (including spoiler rendering fallback logic) — file: `tests/integration/test_notifications.py`
- [ ] T034 [US2] Add metric and alert: notification delivery latency and failure rate — file: `backend/src/lib/observability.py`

## Phase 5: User Story 3 - Aggregated Mail List (Priority: P2)

**Goal**: Provide unified mail listing across linked accounts.
**Independent Test**: Connect two accounts and verify merged list in UI.

- [ ] T035 [US3] Implement message ingestion persistence: store `Message` entries and `otp_flag` metadata — file: `backend/src/services/storage.py`
- [ ] T036 [US3] Implement `GET /api/v1/messages` endpoint with pagination and account origin — file: `backend/src/api/messages.py`
- [ ] T037 [US3] Implement frontend mail list page showing aggregated messages and account origin — file: `frontend/src/pages/MailList.jsx`
- [ ] T038 [US3] Add unit and integration tests for message listing and ordering — file: `tests/integration/test_message_listing.py`
- [ ] T039 [US3] Add indexing/migration task to ensure `provider_message_id` uniqueness per account — file: `backend/alembic/versions/*.py`

## Phase 6: User Story 4 - Open UI in Telegram Web App (Priority: P2)

**Goal**: Provide web-app entry via Telegram, requiring re-auth for sensitive actions.
**Independent Test**: Clicking web-app link in Telegram opens the SPA and requires re-auth for mailbox management.

- [ ] T040 [US4] Implement web-app route and basic SPA shell that loads user mail list — file: `frontend/src/App.jsx`
- [ ] T041 [US4] Implement Telegram web-app link generator and bot menu item — file: `bot/src/commands.js`
- [ ] T042 [US4] Implement session handling that accepts Telegram-derived session but enforces OAuth re-auth for sensitive operations — file: `backend/src/middleware/session.py`
- [ ] T043 [US4] Add tests verifying re-auth gate for sensitive actions (remove account, view full message body) — file: `tests/integration/test_reauth.py`

## Phase 7: Polish & Cross-Cutting Concerns

- [ ] T044 Documentation: update `README.md` and feature docs under `docs/` — file: `docs/telegram-mail-client.md`
- [ ] T045 Security review: run secret-scan, dependency audit, and document findings — file: `security/audit.md`
- [ ] T046 Performance test: simulate notification load and verify delivery SLA — file: `tests/perf/test_notifications_perf.py`
- [ ] T047 Accessibility / UX polish for web-app pages — file: `frontend/src/pages/*`
- [ ] T048 Release: prepare deployment manifests (k8s/terraform) for backend and bot — file: `deploy/`

## Dependencies & Execution Order

- Foundation (Phase 2) MUST be complete before User Story phases begin.
- MVP Recommendation: Implement Phase 3 (US1) and Phase 4 (US2) first.

## Parallel Opportunities

- Tasks marked `[P]` can run in parallel (e.g., initial scaffolding, models and workers).
- Frontend and backend scaffold tasks can run in parallel after project layout is created.

## Implementation Strategy

- MVP first: Complete Phase 1 + Phase 2, then deliver US1 + US2 as an independent MVP.
- Incremental delivery: After MVP, implement US3 and US4, then run polish.

## Summary

- Total tasks: 48
- Tasks per story: US1=8, US2=8, US3=5, US4=4, Setup/Foundational/Polish=23
- Suggested MVP scope: US1 (Connect/Remove) + US2 (Notifications)

