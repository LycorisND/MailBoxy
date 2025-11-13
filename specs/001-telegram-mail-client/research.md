# Research: Telegram Mail Client (Phase 0)

**Feature**: 001-telegram-mail-client
**Created**: 2025-11-14

## Unknowns / NEEDS_CLARIFICATION (from plan)

- Encryption at rest: key management approach for token encryption (KMS vs
  env var).  
- Provider integration patterns: push vs poll for Gmail/Yandex/Outlook.  
- OTP detection heuristics: regex patterns vs ML models and false-positive
  tolerances.

## Decisions

### Language and Framework
Decision: Use Python 3.11 for the backend with FastAPI.
Rationale: FastAPI allows quick development of REST endpoints, good async
support for network-bound tasks (provider APIs, notification dispatch), and
has strong testing and typing support.
Alternatives considered: Node/TypeScript backend (good for JS stack parity),
but Python's ecosystem for mail processing and quick scripting is preferred
for MVP.

### Storage and Token Encryption
Decision: Use PostgreSQL for metadata and S3-compatible object storage for
full message bodies if needed. Encrypt OAuth tokens at rest using an
application-level encryption key provided via environment variable for
initial MVP; recommend KMS (AWS KMS / Google KMS) for production.
Rationale: Postgres is reliable for relational queries and indexing; using
an external KMS for production reduces risk of key leakage.

### Provider Integration
Decision: Use provider APIs where available (Gmail push + API, Outlook
Graph API). For providers without push, implement scheduled polling with
backoff and rate-limit handling.
Rationale: Push is more efficient and real-time for Gmail; polling is
fallback for providers lacking webhook support.

### OTP Detection
Decision: Implement regex-based heuristics initially (common OTP patterns:
6-digit codes, alpha-numeric short tokens), augmented by sender allowlist
for known OTP senders. Record false positive incidents and iterate; ML can
be considered later if heuristics prove insufficient.
Rationale: Regex is simple, explainable, low-risk, and efficient for
initial rollout.

### OTP Delivery Formatting
Decision: Use Telegram's spoiler formatting for OTPs in messages. If the
client does not support spoilers, fall back to showing a masked OTP (last 4
chars) and instruct user to open web-app to view the full code.
Rationale: Respects convenience while reducing accidental visibility.

### Message Persistence & Retention
Decision: Persist full message bodies only when a user opts in per-account.
By default the system stores only snippets and metadata. For persisted
bodies, use a default retention of 30 days; retention is configurable per
account. Implement automated cleanup jobs to enforce retention.
Rationale: Minimizes stored sensitive data by default while allowing users
who need full message storage the option to enable it.

### OAuth scopes & provider notes (Phase 0 concrete)

- Gmail (recommended scopes for read-only OTP notifications):
  - `openid email profile`
  - `https://www.googleapis.com/auth/gmail.readonly` (or `https://www.googleapis.com/auth/gmail.metadata` for metadata-only)
  - Note: Gmail push notifications require a Google Cloud Pub/Sub topic and service account; for MVP polling remains acceptable if Pub/Sub setup is too heavy.

- Microsoft Outlook (Graph API):
  - `openid offline_access profile email`
  - `Mail.Read` (read-only access to mail)
  - Note: use Microsoft identity platform endpoints and request `offline_access` for refresh tokens.

- Yandex / Other providers:
  - Provider-specific mail.read scopes or IMAP OAuth scopes where supported.

Map these scope decisions to the provider registry and `T012/T019-T022`.

### KMS / Key Management (recommendation)

For production, we recommend integrating a managed KMS (AWS KMS, GCP KMS,
or HashiCorp Vault) for application-level token encryption keys and key
rotation. For MVP, an application encryption key provided via env var is
acceptable but must be gated behind CI secret scanning and documented as
temporary. Action items:

- Evaluate KMS providers relative to hosting provider (TASK-R3 -> T051).
- Add `encryption_key_id` tracking (optional/per-account) if per-account
  keying/rotation is required.

### OTP heuristics (initial regex set)

- Candidate regex patterns to start with (tunable):
  - 6-digit numeric: `\b\d{6}\b`
  - 4-6 digit numeric: `\b\d{4,6}\b`
  - Short alphanumeric: `\b[A-Z0-9]{5,8}\b` (case-insensitive)

Collect a small test corpus of sample OTP emails and measure precision/recall
as TASK-R2 (map to T028/T033 for tests).

### Observability: metrics & alert thresholds

- Delivery latency (metric): `notification.delivery_latency_seconds` (histogram)
  - Alert if p95 > 30s for more than 5m.
- Delivery failure rate: `notification.delivery_failures_total` / `notification.attempts_total`
  - Alert if failure rate > 5% over a 5m window, critical if >10%.
- OAuth failure rate and auth errors: alert on spikes (rate > 1% of auth attempts).

Map the required metrics to T017/T034 for implementation and alerting.

## Research Tasks (mapped)

- TASK-R1 -> T012/T019-T022: Enumerate provider endpoints and required scopes; produce short config snippets for each provider.
- TASK-R2 -> T028/T033: Create OTP regex corpus and implement unit tests for detection accuracy.
- TASK-R3 -> T051: Select KMS vendor and document key rotation strategy; implement KMS integration plan.
- TASK-R4 -> T017/T034: Define metrics and alert rules, add to observability backlog.

## Outcome

This document updates the spec with provider scopes, KMS recommendations,
OTP heuristics and concrete metrics, and maps research outputs to tasks in
`tasks.md`. Phase 0 is complete once `research.md`, `data-model.md` and the
OpenAPI contract are validated and pushed to the feature branch.

## Research Tasks (Phase 0 outputs)

- TASK-R1: Evaluate provider APIs for Gmail, Outlook (MS Graph), Yandex; list
  required scopes and recommended endpoints.
- TASK-R2: Draft initial OTP regex set and create test corpus from sample
  messages to validate precision/recall.
- TASK-R3: Define encryption key management plan for MVP vs production (env
  var vs KMS).
- TASK-R4: Define observability metrics and alert thresholds for notification
  delivery and auth failures.

## Consolidated findings

- Decision: Python + FastAPI, Postgres, S3 optional, Token encryption via
  app key for MVP, KMS for production, OTP using regex heuristics, Gmail
  push when possible.

