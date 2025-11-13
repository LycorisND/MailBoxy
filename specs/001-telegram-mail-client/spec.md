# Feature Specification: Telegram Mail Client

**Feature Branch**: `001-telegram-mail-client`  
**Created**: 2025-11-14  
**Status**: Draft  
**Input**: Product Vision: почтовый клиент где можно подключить все почтовые ящики и получать письма прямо в телеграм.

User Stories provided:
- Как пользователь, я хочу открывать интерфейс почтового клиента в telegram web app.
- Как пользователь, я хочу возможность подключать и\или удалять почтовые ящики через OAuth.
- Как пользователь, я хочу видеть всю почту со всех подключенных ящиков в списке почты.
- Как пользователь, я хочу получать уведомления когда поступают новые письма (телеграм бот отправляет сообщение в чат). Если почта содержит одноразовый код для авторизации, то бот отправляет сообщение, в остальных случаях сообщения отправляются без звука.

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - Connect / Remove Mailboxes (Priority: P1)

As a user I can connect a mailbox account (Gmail, Yandex, Outlook, etc.) via
OAuth and remove it later. The connection flow MUST guide the user through
provider consent and clearly show requested scopes.

Why this priority: Without mailbox connections the product has no input
source — connecting via OAuth is required for all subsequent features.

Independent Test: Use a test account to complete OAuth flow for one provider
and verify tokens are stored encrypted and the mailbox appears in the
account list.

Acceptance Scenarios:
1. **Given** an unauthenticated user, **When** they initiate "Add account",
   **Then** user is redirected to provider OAuth consent and, after consent,
   the account appears in MailBoxy.
2. **Given** an existing connected account, **When** user chooses "Remove",
   **Then** MailBoxy revokes stored tokens and removes the account from the
   list.

---

### User Story 2 - Notifications in Telegram (Priority: P1)

As a user I receive Telegram notifications when new mail arrives. If the
message contains a one-time code (OTP) the bot MUST send a high-priority
notification; otherwise notifications are delivered silently by default.

Why this priority: Immediate notification is the core value proposition —
users need timely alerts for OTPs and general awareness of incoming mail.

Independent Test: Send test emails (with and without OTP) to a connected
account and verify Telegram messages match priority rules and content
sanitization rules.

Acceptance Scenarios:
1. **Given** a new incoming email containing an OTP, **When** MailBoxy
   processes it, **Then** the Telegram bot sends an audible/high-priority
   notification with minimal identifying info and the OTP.
2. **Given** a new incoming email without OTP, **When** MailBoxy processes
   it, **Then** the Telegram bot sends a silent notification containing
   sender and subject only.

---

### User Story 3 - Aggregated Mail List (Priority: P2)

As a user I want to see a unified list of messages from all connected
accounts so I can browse recent mail without switching accounts.

Why this priority: Aggregation improves usability but can be staged after
basic notification/connectivity features.

Independent Test: Connect two test accounts, send mail to both, and verify
the web UI lists messages from both accounts sorted by received time.

Acceptance Scenarios:
1. **Given** two connected accounts with new messages, **When** user opens
   the mail list, **Then** entries from both accounts appear merged and
   ordered by message date.

---

### User Story 4 - Open UI in Telegram Web App (Priority: P2)

As a user I want to open the full mail client UI inside the Telegram web
app (web-app) to read and manage messages with a richer interface.

Why this priority: The web-app provides full functionality beyond
notifications; it is important but secondary to connection and notification
flows for initial MVP.

Independent Test: From Telegram bot, click "Open MailBoxy" and confirm the
web-app loads, shows authenticated user's mail list, and allows read actions.

Acceptance Scenarios:
1. **Given** a connected user, **When** they click the web-app link in
   Telegram, **Then** the `mailboxy.app` web UI opens inside Telegram web-app
   and shows their aggregated mail.

---

### Edge Cases

- Provider OAuth consent denied (user cancels) — app must show clear retry
  path and not create a partial account record.
- Token refresh failure or revocation by provider — app must detect and
  mark account as disconnected, and notify the user with remediation steps.
- Multiple accounts with overlapping senders — UI must show account origin
  for each message to avoid confusion.
- Rate limits from providers — retries with backoff and postponed syncs.

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: System MUST [specific capability, e.g., "allow users to create accounts"]
- **FR-002**: System MUST [specific capability, e.g., "validate email addresses"]  
- **FR-003**: Users MUST be able to [key interaction, e.g., "reset their password"]
- **FR-004**: System MUST [data requirement, e.g., "persist user preferences"]
- **FR-005**: System MUST [behavior, e.g., "log all security events"]

<!-- Constitution-mandated functional requirements examples -->
- **FR-AUTH-001**: System MUST integrate mailbox providers via OAuth or
  equivalent secure delegation flows; credentials MUST NOT be stored.
- **FR-NOTIFY-001**: Notification delivery MUST implement OTP-first policy:
  OTP-containing messages MUST trigger high-priority alerts; other mail
  notifications MUST be silent by default.
- **FR-DOMAIN-001**: Web UI and OAuth redirect URIs MUST use the official
  domain `mailboxy.app` unless an explicit exception is approved.

*Example of marking unclear requirements:*

- **FR-006**: System MUST authenticate users via [NEEDS CLARIFICATION: auth method not specified - email/password, SSO, OAuth?]
- **FR-007**: System MUST retain user data for [NEEDS CLARIFICATION: retention period not specified]

### Key Entities *(include if feature involves data)*

- **[Entity 1]**: [What it represents, key attributes without implementation]
- **[Entity 2]**: [What it represents, relationships to other entities]

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: [Measurable metric, e.g., "Users can complete account creation in under 2 minutes"]
- **SC-002**: [Measurable metric, e.g., "System handles 1000 concurrent users without degradation"]
- **SC-003**: [User satisfaction metric, e.g., "90% of users successfully complete primary task on first attempt"]
- **SC-004**: [Business metric, e.g., "Reduce support tickets related to [X] by 50%"]
