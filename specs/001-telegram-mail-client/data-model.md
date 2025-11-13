# Data Model: Telegram Mail Client

**Feature**: 001-telegram-mail-client
**Created**: 2025-11-14

## Entities

### User
- id: uuid
- telegram_user_id: string (unique)
- display_name: string
- email: string (optional)
- preferences: jsonb (per-account notification prefs)
- created_at, updated_at

### MailboxAccount
- id: uuid
- user_id: uuid (FK -> User)
- provider: enum (gmail, outlook, yandex, other)
- provider_account_id: string
- display_name: string
- encrypted_access_token: bytes
- encrypted_refresh_token: bytes (optional)
- scopes: text[]
- sync_status: enum (connected, disconnected, error)
- last_sync_at: timestamp
- created_at, updated_at
- persist_messages: boolean (default: false)  # per-account opt-in for full body storage
- retention_days: integer (nullable, default: 30)  # retention period for persisted bodies

### Message
- id: uuid
- mailbox_account_id: uuid (FK -> MailboxAccount)
- provider_message_id: string
- sender: string
- recipients: text[]
- subject: string
- snippet: text
- received_at: timestamp
- stored_body_ref: string (object store key) (optional)
- otp_flag: boolean
- otp_masked: string (last 4 chars or null)
- processing_metadata: jsonb (detection heuristics, raw headers)
- created_at, updated_at

### Notification
- id: uuid
- user_id: uuid
- message_id: uuid
- priority: enum (high, silent)
- delivered_at: timestamp (nullable)
- delivery_status: enum (pending, sent, failed)
- telegram_chat_id: string
- created_at, updated_at

## Relationships
- User 1..* MailboxAccount
- MailboxAccount 1..* Message
- Message 1..* Notification

## Validation rules
- `Message.provider_message_id` unique per `mailbox_account_id`.
- `MailboxAccount.provider_account_id` uniqueness per provider per user.
- `otp_masked` present only if `otp_flag` is true.

## State transitions
- MailboxAccount.sync_status: connected -> error -> disconnected (on
  revocation)
- Notification.delivery_status: pending -> sent | failed; retry limited
  with exponential backoff

