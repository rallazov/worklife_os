# Data Retention & Storage Matrix (Current)

This is a **safe-to-commit** summary of what data enters the product, where it lives, and how long it persists **as implemented today**.

If you want a **private** version with real example payloads, specific vendors, account IDs, or screenshots, keep that in your Obsidian vault (or under an ignored folder like `docs/private/`).

## Storage tiers
- **Hot (DB)**: persisted in PostgreSQL/SQLite and used for product behavior.
- **Warm (on-demand)**: fetched when needed (usually from Gmail); not guaranteed persisted.
- **Cold (file/object)**: local disk today; should be object storage later.

## Matrix

| Data / input | Where it comes from | Hot (DB) | Warm (on-demand) | Cold (files) | Retention (current) | Notes / sensitivity |
|---|---|---|---|---|---|---|
| User account & profile | Signup / onboarding | `users` (email, display_name, timezone, zip_code, local_context, prep_times, onboarding fields) | — | — | Until account deleted | **PII**: email, zip code |
| Auth identities | OAuth/login providers | `auth_identities` | — | — | Until account deleted | Links provider subject to user |
| OAuth tokens (Gmail) | OAuth consent | `users.google_token`, `mailbox_connections.token_json` (**encrypted**) | — | — | Until disconnect/revoke | **Secret**: encrypted at rest via Fernet (`ENCRYPTION_KEY`) |
| OAuth state (transient) | OAuth handshake | `oauth_states` | — | — | No cleanup job today | Should be periodically pruned |
| Domain configs (“modes”) | System defaults + user edits | `domain_configs` | — | — | Forever | Can contain user-provided labels/icons/colors |
| Email index (metadata) | Gmail sync | `emails` (message_id, snippet, subject, from/to, labels, internal_date, etc.) | — | — | Forever | **PII** in headers/snippets; scales by rows |
| Email body cache | User opens email drawer | `emails.body_text/body_html` once fetched | Gmail API fetch (first time) | — | Forever (no TTL today) | Consider TTL or “store none” policy later |
| Attachment metadata | On-demand email fetch | `emails.attachments` (filename, mime_type, size, attachment_id) | — | — | Forever | Filenames can be sensitive |
| Attachment bytes | User downloads attachment | — | Proxied from Gmail API (`/email/attachments/...`) | — | Gmail retention | Server does **not** persist bytes (in transit only) |
| Work items (canonical tasks/notes/apps) | Derived from emails + manual create | `job_applications` (`WorkItem`) | — | — | Forever | Your “system of record” |
| Work item notes | User edits | `job_applications.notes` | — | — | Forever | Can include sensitive content if user types it |
| AI-derived enrichment (emails) | AI triage/enrichment | Stored as **derived fields** on `WorkItem` (title/summary/domain/status/etc.) | — | — | Forever | Raw prompts/responses not persisted by default |
| Document/image uploads (raw) | `/documents/analyze` | — | — | Local `uploads/` (relative path at runtime) | `DOCUMENT_UPLOAD_RETENTION_HOURS` (default **168h / 7d**), best-effort cleanup | **High risk**: receipts, handwriting, IDs |
| Document/image extraction results | Vision extraction response | Persisted only if user creates a WorkItem (title/summary/domain/notes) | — | — | Forever (if saved as WorkItem) | Consider persisting full extracted text into `WorkItem.meta` later |
| Client-side preferences | Browser | — | — | `localStorage` (e.g., first-visit UI hints) | Until user clears | Don’t store secrets in localStorage |

## What NOT to commit (even in a private repo)
- Real user emails, screenshots, invoices/receipts, exports, or database dumps.
- Any secrets: `OPENAI_API_KEY`, `ENCRYPTION_KEY`, OAuth client secrets, `DATABASE_URL`, JWT signing keys.
- Any token JSON (even encrypted) captured in logs or docs.

## Practical hygiene
- Keep “real data” docs in Obsidian or an ignored folder like `docs/private/`.
- Ensure upload folders are ignored (`uploads/`, `backend/uploads/`).
- When adding docs/examples, use placeholders like `user@example.com`, `COMPANY_X`, and `ATTACHMENT_ID_REDACTED`.
