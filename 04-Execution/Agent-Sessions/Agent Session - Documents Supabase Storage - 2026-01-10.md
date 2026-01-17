# Agent Session
tags: #agent

## Goal
Replace Documents page local upload storage with Supabase Storage and add a secure processing flow.

## Topic
Documents page storage + processing (Supabase Storage, FastAPI endpoints, minimal UI wiring)

## Prompt(s)
- Implement a production-safe document upload flow for the Documents page using Supabase Storage.
- Replace local uploads with a private bucket + documents table + backend processing endpoints.
- Keep UI changes minimal (wire the flow only) and keep accepted types to PDF/JPG/PNG.

## Output Summary
- Storage: private Supabase bucket `documents` with path `auth_uid/document_id/filename`.
- Data model: `documents` table with status tracking and extracted_json.
- Backend: /documents/init, /documents/{id}/process, /documents/{id}, /documents/{id}/download-url.
- Add DELETE endpoint for document removal.
- Security: JWT required; RLS deny direct client access; storage policy scoped to auth.uid(); service role backend-only.
- Frontend: init -> upload -> process -> poll -> download.
- Retention: optional expires_at + cleanup command.

## Implementation Plan
- SQL migration for documents table (status enum + timestamps).
- Create Supabase Storage bucket and policies.
- Implement backend endpoints and idempotent processing.
- Wire Documents page upload flow and polling.
- Add cleanup script and verification checklist.

## Best Practices
- Enforce file type and size on both client and server.
- Sanitize filenames before storage.
- Do not expose service role key to the client.
- Ensure processing is idempotent to avoid duplicate WorkItems.

## Decisions
- Accepted:
  - Documents page accepts PDF/JPG/PNG only.
  - Max upload size set to 10MB.
  - Backend owns documents table writes.
  - Signed download URLs (TTL 5 minutes).
  - WorkItems store document_id for traceability.
  - Allow retry when status is failed.
  - Default expires_at is null (no automatic expiry for MVP).
  - Orphan cleanup threshold set to 6 hours (config ORPHAN_UPLOAD_TTL_HOURS=6).
- Rejected:
  - Third-party upload vendors.
  - Local disk storage for uploads.
  - DOCX/CSV/TXT support on Documents page.

## Next Actions
- [ ] Create SQL migration and storage policies.
- [ ] Implement backend endpoints.
- [ ] Add DELETE /documents/{id} and retry for failed processing.
- [ ] Add orphan cleanup for status=uploaded older than X hours.
- [ ] Add rate limiting for /documents/init (e.g., 10/min).
- [ ] Wire frontend upload + polling flow.
- [ ] Add cleanup command and document retention behavior.

## Links / References
- docs/DOCUMENTS_SUPABASE_STORAGE_PLAN.md
- docs/DATA_RETENTION_MATRIX.md
- frontend/src/pages/DocumentsPage.tsx
- backend/main.py (document processing flow)
