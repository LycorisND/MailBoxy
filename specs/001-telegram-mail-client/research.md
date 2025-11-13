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

