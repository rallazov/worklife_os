# System Overview
tags: #spec

## Components
- Frontend (React): Today / Workbench / Insights
- Backend (FastAPI): auth, mailbox connections, sync, extraction pipeline
- DB (Postgres recommended): users, identities, mailboxes, work_items, emails, stats

## Key separations (core architecture)
- Identity login ≠ mailbox OAuth connection
- Work items are system of record; AI assists, rules enforce determinism
