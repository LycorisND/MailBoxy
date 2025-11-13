<!--
Sync Impact Report

- Version change: TEMPLATE -> 1.0.0
- Modified principles: (added)
	- Privacy & Data Minimization
	- OAuth-First & Least Privilege
	- Notification Policy (OTP-first)
	- Telegram Integration & Web-app UX
	- Reliability, Observability & Testing
- Added sections:
	- Security & Privacy Constraints
	- Development Workflow
- Removed sections: none
- Templates updated: ✅ .specify/templates/plan-template.md, ✅ .specify/templates/spec-template.md
- Templates pending manual review: ⚠ .specify/templates/agent-file-template.md, ⚠ .specify/templates/checklist-template.md
- Follow-up TODOs:
	- TODO(RATIFICATION_DATE): confirm official ratification date
	- Review `agent-file-template.md` for project-specific tech extraction
	- Ensure `spec-template.md` FRs align with feature specs
-->

# MailBoxy Constitution

## Core Principles

### Privacy & Data Minimization (NON-NEGOTIABLE)
The system MUST collect and retain the minimal data necessary to provide
functionality. MailBoxy MUST NOT store user passwords; authentication is
handled via OAuth flows only. Persistent storage of tokens MUST be encrypted
at rest and access-restricted. Retention periods for message content and
metadata MUST be documented per-account; defaults MUST be conservative.

Rationale: Mail clients handle sensitive personal data. Minimizing what is
stored and encrypting it reduces risk and regulatory exposure.

### OAuth-First Authentication & Least Privilege
All mailbox integrations (Gmail, Yandex, Outlook, etc.) MUST use OAuth or
equivalent secure delegation flows. Requested scopes MUST follow least
privilege: request only the scopes needed for notification and message
preview. Token refresh and revocation flows MUST be handled securely and
documented. Integrations MUST use `mailboxy.app` as the official application
domain and provide clear consent screens.

Rationale: OAuth reduces credential leakage and aligns with provider
requirements; least-privilege limits blast radius if a token is compromised.

### Notification Policy (OTP-First)
Notifications delivered via the Telegram bot are governed by a strict policy:
- Messages containing one-time/password codes (OTPs) or explicit auth codes
	MUST trigger high-priority notifications (audible or otherwise prominent).
- All other incoming mail notifications MUST be delivered as silent messages
	(no sound) by default. Users MAY configure per-account preferences.
- Message content forwarded to Telegram MUST avoid exposing sensitive data
	beyond the minimum required to identify the sender and subject.

Rationale: Prioritize urgent security messages while reducing notification
fatigue and accidental exposure of content on notification channels.

### Telegram Integration & Web-app UX
MailBoxy is designed as a Telegram-first experience. The Telegram bot MUST
provide secure notifications and a web-app entry point that opens the full
mail client UI inside the Telegram web-app. The web UI MUST be responsive,
secure (HTTPS on `mailboxy.app`), and require explicit user re-authentication
for sensitive actions.

Rationale: Tight integration with Telegram gives users a single control
surface for notifications and quick access while preserving a richer web
interface for full mail interactions.

### Reliability, Observability & Testing
The project MUST implement structured logging, metrics (latency, message
processing rate, delivery errors), and alerting for critical failures.
Automated tests are mandatory for authentication flows, notification
filtering (OTP detection), and critical message-processing paths. Security
reviews and compliance checks MUST be performed regularly.

Rationale: Observability permits rapid detection and response; testing
prevents regressions in security-sensitive areas.

## Security & Privacy Constraints
All third-party dependencies that handle authentication or message parsing
MUST be evaluated for security posture. Static secrets are forbidden in
repository code; configuration MUST come from environment variables or a
secure secrets store. Any telemetry or analytics collected MUST be
documented and opt-out enabled.

## Development Workflow
- All features MUST include a spec (`specs/<feature>/spec.md`) with
	measurable acceptance criteria and privacy considerations.
- Constitution Check: every plan (`plan.md`) MUST include a short section
	confirming compliance with the Privacy, OAuth, and Notification principles.
- Tests for security-critical flows (OAuth, token storage, OTP-detection)
	MUST be included and executed in CI.
- Code review: at least one maintainer review is required for production
	changes; security-affecting changes require a designated security reviewer.

## Governance
Amendments to this constitution require a documented proposal (PR) and
approval by a majority of project maintainers. Major governance changes that
remove or redefine core principles MUST increment the MAJOR version. New
principles or material additions increment MINOR. Clarifications and typo
fixes increment PATCH.

- Amendment process: create PR against `.specify/memory/constitution.md`,
	include migration notes and impact assessment, and obtain maintainer
	approvals.
- Compliance review: a quarterly review of security controls and privacy
	retention policies is REQUIRED.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE) | **Last Amended**: 2025-11-14

