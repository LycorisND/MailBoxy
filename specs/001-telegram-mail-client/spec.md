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

## Clarifications

### Session 2025-11-14

- Q: Authentication / account linking between Telegram and MailBoxy → A: Require explicit linking via OAuth; web-app may reuse Telegram session for convenience but MUST require OAuth re-authentication or re-authorization for sensitive actions.
- Q: Notification delivery destination → A: Deliver notifications to the user's private Telegram chat only by default; no group/channel delivery unless explicitly requested in a future opt-in flow.
 - Q: OTP content in notifications → A: Include the full OTP in high-priority Telegram notifications but formatted as hidden/spoiler text (Telegram spoiler markup). If the recipient client does not support spoiler formatting, fall back to showing a masked OTP (last 4 characters).
 - Q: Message body retention policy → A: Persist full message bodies only if the user opts in per-account; by default only snippets are stored. Default retention for persisted bodies: 30 days (user-configurable per-account).

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
  notification with minimal identifying info and the full OTP formatted as
  hidden/spoiler text using Telegram's supported markup. If the Telegram
  client does not render spoilers, fall back to displaying a masked OTP
  (last 4 characters) and prompt the user to open the web-app for the full
  code.
2. **Given** a new incoming email without OTP, **When** MailBoxy processes
   it, **Then** the Telegram bot sends a silent notification containing
   sender and subject only.

Additional constraint: Notifications are delivered to the user's private
Telegram chat by default. Group or channel delivery is NOT supported in the
initial MVP and requires an explicit opt-in flow and additional consent.

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

Additional constraint: the web-app MAY accept a Telegram-derived session to
provide a convenient single-click entry, BUT sensitive actions (adding/removing
mailboxes, viewing full message bodies beyond snippets, changing notification
preferences) MUST require explicit OAuth re-authentication or re-confirmation
to mitigate impersonation and session risks.

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

- **FR-OAUTH-001**: OAuth-first integration: The system MUST integrate
  mailbox providers (Gmail, Outlook, Yandex, etc.) via OAuth or equivalent
  secure delegation flows. Tokens MUST be stored encrypted at rest and
  never committed to repository code. Mailbox linking and management actions
  (add/remove accounts, token refresh/revocation) MUST use explicit OAuth
  flows; the web-app MAY accept a Telegram-derived session for convenience
  but sensitive operations MUST require re-authentication or re-authorization.

- **FR-NOTIFY-001**: Notification policy: Notification delivery MUST
  implement the OTP-first policy: OTP-containing messages MUST trigger
  high-priority alerts; non-OTP messages MUST be delivered silently by
  default. Notifications MUST be sent to the user's private Telegram chat
  by default; group/channel delivery is out-of-scope for MVP and requires
  explicit opt-in.

- **FR-OTP-001**: OTP formatting and delivery: For OTP-containing emails,
  Telegram notifications MUST include the full OTP formatted as hidden/
  spoiler text when supported by the client. If spoilers are unsupported,
  notifications MUST fall back to a masked OTP (last 4 characters) and
  provide instructions to retrieve the full code via the web-app.

- **FR-RET-001**: Message persistence & retention: Full message bodies MAY
  be persisted only when the user opts in per-account (`persist_messages`).
  By default only snippets and metadata are stored. Persisted bodies MUST
  have a configurable retention period (default: 30 days) enforced by
  automated cleanup jobs.

- **FR-DOMAIN-001**: Domain and redirect URIs: Web UI and OAuth redirect
  URIs MUST use the official domain `mailboxy.app` in production. The plan
  MUST include tasks to provision DNS and TLS and register redirect URIs
  with each provider.

- **FR-RET-001**: Message persistence MUST be opt-in per-account. By
  default the system stores only message snippets and metadata; if a user
  enables message persistence for an account, full message bodies may be
  stored with a default retention of 30 days. Retention period MUST be
  configurable per-account and enforced by automated cleanup jobs.

*Example of marking unclear requirements:*

- **FR-006**: System MUST authenticate users via [NEEDS CLARIFICATION: auth method not specified - email/password, SSO, OAuth?]
- **FR-007**: System MUST retain user data for [NEEDS CLARIFICATION: retention period not specified]

### Key Entities *(include if feature involves data)*

- **User**: id, Telegram user id, display name, preferences (per-account
  notification settings), linked mailbox account ids.
- **MailboxAccount**: provider (Gmail/Yandex/Outlook), account id, display
  name, encrypted access/refresh tokens, sync status, last sync timestamp.
- **Message**: provider message id, mailbox account id, sender, recipients,
  subject, snippet, stored content reference (if persisted), received time,
  OTP flag (boolean), processing metadata.
- **Notification**: notification id, message id, user id, priority (high/
  silent), delivery timestamp, delivery status, recipient chat id.

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: OAuth connect flow: A user can complete mailbox linking via
  OAuth within 3 minutes in 95% of successful attempts (measured in tests).
- **SC-002**: Notification latency: Telegram notifications for OTP emails
  are delivered within 30 seconds in 95% of test deliveries under normal
  operating conditions.
- **SC-003**: OTP detection accuracy: OTP detection heuristics achieve >=95%
  precision on a representative test corpus (minimize false positives).
- **SC-004**: Aggregation correctness: The aggregated mail list shows
  messages from multiple accounts merged and correctly ordered by received
  time in 99% of test cases.
