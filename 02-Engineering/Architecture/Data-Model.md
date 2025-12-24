# Data Model
tags: #spec

## Core tables (high-level)
- User
- AuthIdentity (google/password/microsoft)
- MailboxConnection (gmail/outlook tokens + status)
- Email (headers/snippet/body optional)
- WorkItem (type/domain/state/status/company/entity metadata)

## Notes
Keep settings separate (UserSettings table or JSON) to avoid User becoming a junk drawer.
