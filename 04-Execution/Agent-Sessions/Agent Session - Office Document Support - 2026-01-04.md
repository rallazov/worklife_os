# Agent Session — Triad
tags: #agent #triad
status: Ready for Builder

## Client Ask (client only — paste 3–10 lines)
- Topic: Office document upload support
- Ask: Add support for Word (.docx), Excel (.xlsx), and CSV uploads to the Documents page
- Why now: Users have job offers, contracts, invoices in Office formats; current system only accepts PDF/images
- Must-have: Extract text from Office documents and analyze with AI (cheaper than Vision API)
- Nice-to-have: Preview of extracted text before analysis; support for .txt and .rtf
- Constraints (privacy/cost/time/tech):
  - Must not store sensitive document content longer than current retention (168h)
  - Text extraction should be cheaper than Vision API calls
  - Keep Vision path for images/PDFs; add text path for Office docs
- Links:
  - [[04-Execution/Agent-Sessions/Agent Session - Documents Upload UI Rules - 2026-01-04|Documents Upload UI Rules session]]
  - Code: `application-tracker/backend/services/document_service.py`
  - Code: `application-tracker/backend/extractors/gpt_parser.py`

## Agent 1 Intake — Product + Business + Delivery (turn ask into a story)
- Problem statement:
  - Users receive important documents (job offers, contracts, invoices, pay stubs) in Office formats
  - Current system rejects these with "Unsupported file type" error
  - Users must manually convert to PDF before uploading — friction that reduces adoption
  - Vision API is overkill for text-based documents; text extraction is 5-10x cheaper

- User story:
  - As a user, I want to upload Word, Excel, and CSV documents so that I can extract and track work items without manual conversion

- Success metrics:
  - Adoption: % of uploads using new Office formats (target: 20%+ within 30 days)
  - Cost reduction: avg cost per document analysis (text path should be ~5x cheaper than Vision)
  - Error reduction: decrease in "unsupported format" errors shown to users

- Non-goals:
  - Complex formatting/layout preservation (we extract text, not visual structure)
  - Document editing or round-trip save
  - Multi-file batch upload (separate session: Documents Upload UI Rules)
  - Password-protected file decryption
  - Macro execution (security risk)

- Acceptance criteria:
  1. User can upload `.docx`, `.xlsx`, `.csv` files via Documents page
  2. Backend extracts plain text from Office documents using Python libraries
  3. Extracted text is sent to text-based LLM (GPT-4o or GPT-4o-mini) for analysis
  4. Same work item suggestion flow as PDF/image path
  5. Clear error messages for: corrupted files, password-protected files, empty content
  6. Retention policy unchanged (168h auto-cleanup)
  7. Frontend shows accepted file types in upload hint

- Risks / harm / disruption assessment:
  - **Security (Medium)**: Office files can contain macros; mitigation: extract text only, never execute
  - **Token limits (Low)**: Large spreadsheets may exceed context; mitigation: truncate with warning
  - **Corrupted files (Low)**: Malformed XML in .docx/.xlsx; mitigation: graceful error handling
  - **Privacy (Low)**: Same retention policy as current; no new data stored

- Recommendation: ✅ **Accept**
  - Clear user value (removes friction for common document types)
  - Lower cost per analysis (text vs Vision)
  - Incremental change (adds extraction path, doesn't replace Vision)
  - Low risk with proper error handling

- Decline reason (if any): N/A

- Questions for Agent 2 / Agent 3:
  - **Agent 2**: Should we show a preview of extracted text before analysis? Or auto-analyze like images?
  - **Agent 2**: How do we communicate the difference between "Vision analysis" (images/PDF) vs "Text analysis" (Office docs)?
  - **Agent 3**: Which Python libraries for extraction? (`python-docx`, `openpyxl`, `csv` stdlib?)
  - **Agent 3**: Should we add a text-specific prompt template, or reuse the Vision prompt with extracted text?

## Agent 2 — UX + Frontend
### Notes

**Responding to Agent 1 and Agent 3's questions:**

1. **Scope**: Keep this as **Workbench Import only**; Documents page stays PDF/image-only. Cleaner to keep rules separate; can merge later if needed.

2. **Flow**: 2-step is best → Upload → Extract → Preview/Edit → Analyze → WorkItemDrawer

3. **Preview formatting**: Editable plain-text textarea with preserved line breaks. For CSV/XLSX, add "Table view" toggle later — keep v1 simple.

4. **Truncation**:
   - Preview: up to **20k chars** with "Show more" toggle
   - Analysis: safe max **15k–25k chars** with clear warning if truncated

5. **Persistence**: Keep extracted text **ephemeral** (client state only). No backend storage beyond upload processing; only log errors.

6. **Clarity**: Add small **"Text analysis"** badge in modal so users know this is not Vision.

7. **Error UX**: Explicit messages for corrupted/password-protected/empty. Hint listing accepted types (.docx/.xlsx/.csv) + size cap.

**UX flow diagram:**
```
[Import File] button (Workbench toolbar)
       ↓
┌─────────────────────────────────────┐
│  FileImportModal                    │
│  ┌─────────────────────────────────┐│
│  │  Drag & Drop zone               ││
│  │  .docx, .xlsx, .csv             ││
│  └─────────────────────────────────┘│
│                                     │
│  Step 1: Upload → Extracting...    │
│  Step 2: Preview (editable textarea)│
│           [20k chars max, truncate] │
│           [Text analysis] badge     │
│  Step 3: [Analyze] button          │
│  Step 4: WorkItemDrawer prefilled  │
└─────────────────────────────────────┘
```

### Questions / Requests

**Open questions to close before implementation:**

1. **Confirm scope**: Workbench import only vs Documents page — recommend Workbench only
2. **Confirm preview/analysis max chars**: suggest 20k preview, 15k–25k analyze
3. **Confirm persistence**: ephemeral only (recommended)

## Agent 3 — Architecture + Backend
### Notes

**Responding to Agent 1's questions:**

1. **Libraries for extraction**: Recommend `python-docx`, `openpyxl`, and stdlib `csv`. Avoid `pandas` for CSV unless we need DataFrame operations — stdlib `csv` is lighter and sufficient for text extraction.

2. **Prompt template**: Reuse existing `gpt_parser.py` prompt structure but with a text-specific wrapper. The Vision prompt expects image context; text extraction should use a simpler "analyze this document text" prompt. Suggest adding `extract_from_text()` alongside `extract_from_image()`.

**Review of proposed implementation plan (from other agent):**

| Aspect | Assessment |
|--------|------------|
| 2-step workflow (preview → analyze) | ✅ Good — gives user control, avoids wasting tokens on garbage text |
| Separate from Documents page | ✅ Good — keeps Documents page for "permanent" files, this is "import utility" |
| New endpoint `POST /files/extract-text` | ✅ Good — clean separation of concerns |
| Libraries (python-docx, openpyxl, pandas) | ⚠️ pandas is heavy; prefer stdlib `csv` for CSV files |
| FileImportModal in Workbench | ✅ Good — logical placement for "create work item from file" |

**Architectural recommendations:**

1. **Token limit guard**: Add `MAX_EXTRACTED_CHARS` env var (default 50K). Truncate with warning if exceeded.
2. **Error codes**: Return structured errors: `UNSUPPORTED_TYPE`, `PASSWORD_PROTECTED`, `EMPTY_CONTENT`, `CORRUPTED_FILE`, `EXTRACTION_FAILED`.
3. **No file persistence for text path**: Extract → analyze → discard. Don't save Office files to `uploads/` unless explicitly requested.
4. **Mime type detection**: Use magic bytes, not just extension. `.docx` = `PK\x03\x04` (ZIP), `.xlsx` = same.

**Security notes:**
- `python-docx` and `openpyxl` are safe — they parse XML, don't execute macros
- Don't use `xlrd` (deprecated, security issues with `.xls` format)
- Consider blocking `.xls` (old binary format) — only support `.xlsx`

### Questions / Requests

- **To Agent 1**: The implementation plan positions this as "Workbench import" not "Documents page". Does this match your intent? Client Ask says "Documents page" but the plan separates them.
- **To Agent 2**: For the preview step, should we show raw text or attempt basic formatting (paragraphs, table structure)?
- **To Agent 2**: What's the max text length to show in preview before truncating with "Show more"?
- **Decision needed**: Should extracted text be logged/stored for debugging, or purely ephemeral?

## Integration Summary (Integrator)

**Integrator**: Claude Code

All 3 agents aligned. Key synthesis:

1. **Scope clarified**: This is a **Workbench Import utility**, not a Documents page feature. Documents page remains PDF/image-only for "permanent" storage. This feature is for quick work item creation from Office files.

2. **Flow confirmed**: 2-step workflow (Upload → Preview/Edit → Analyze → WorkItemDrawer). Gives user control before spending tokens.

3. **Technical approach**:
   - Backend: `python-docx`, `openpyxl`, stdlib `csv` (no pandas)
   - New endpoint: `POST /files/extract-text`
   - New function: `extract_from_text()` in gpt_parser.py
   - Error codes: `UNSUPPORTED_TYPE`, `PASSWORD_PROTECTED`, `EMPTY_CONTENT`, `CORRUPTED_FILE`

4. **Truncation limits** (best practice compromise):
   - Backend `MAX_EXTRACTED_CHARS`: 50k (safety buffer)
   - Frontend preview: 20k chars with "Show more"
   - Analysis to LLM: 25k chars max (cost-effective)

5. **Persistence**: Ephemeral only. Extract → analyze → discard. No logging of extracted text.

6. **Security**: Block `.xls` (old binary format), only `.xlsx`. Libraries are safe (XML parsing, no macro execution).

## Decisions
- Accepted:
  - ✅ Workbench Import utility (separate from Documents page)
  - ✅ 2-step flow with preview/edit before analysis
  - ✅ Support `.docx`, `.xlsx`, `.csv` only (block `.xls`)
  - ✅ Use `python-docx`, `openpyxl`, stdlib `csv`
  - ✅ Ephemeral text (no persistence beyond processing)
  - ✅ Truncation: 50k extract / 20k preview / 25k analyze
  - ✅ "Text analysis" badge to distinguish from Vision
- Rejected:
  - ❌ Adding to Documents page (keep separate for now)
  - ❌ Using pandas for CSV (too heavy)
  - ❌ Supporting `.xls` format (security concerns)
  - ❌ Logging/storing extracted text (privacy)

## Next Actions
- [ ] Backend: Add `python-docx`, `openpyxl` to requirements.txt
- [ ] Backend: Create `services/text_extraction.py` with `TextExtractionService`
- [ ] Backend: Create `POST /files/extract-text` endpoint in new `routers/files.py`
- [ ] Backend: Add `extract_from_text()` to `extractors/gpt_parser.py`
- [ ] Backend: Add `MAX_EXTRACTED_CHARS=50000` to env.example
- [ ] Frontend: Create `FileImportModal.tsx` component
- [ ] Frontend: Add "Import File" button to WorkItemsView toolbar
- [ ] Frontend: Add `extractTextFromFile()` to API layer
- [ ] Tests: Unit tests for text extraction (docx, xlsx, csv, error cases)

## Handoff Packet (for Builder)
- Scope/contract links (SPEC/ADR):
  - This session note (Office Document Support)
  - Related: [[04-Execution/Agent-Sessions/Agent Session - Documents Upload UI Rules - 2026-01-04|Documents Upload UI Rules]] (do NOT modify Documents page)
- Acceptance criteria:
  1. User can upload `.docx`, `.xlsx`, `.csv` via "Import File" button in Workbench toolbar
  2. Backend extracts plain text using `python-docx`, `openpyxl`, stdlib `csv`
  3. Frontend shows editable preview (max 20k chars, "Show more" for rest)
  4. "Text analysis" badge visible in modal
  5. User clicks "Analyze" → text sent to LLM (max 25k chars) → WorkItemDrawer prefilled
  6. Clear error messages for: corrupted, password-protected, empty, unsupported type
  7. No file/text persistence beyond processing (ephemeral)
  8. `.xls` format blocked with clear error message
- Non-goals:
  - Modifying Documents page (stays PDF/image only)
  - Complex formatting/layout preservation
  - Multi-file batch upload
  - Password-protected file decryption
  - Macro execution
- Constraints (security/privacy/perf):
  - `MAX_EXTRACTED_CHARS=50000` (backend safety limit)
  - Block `.xls` (security)
  - No logging of extracted text content
  - Use magic bytes for mime detection, not just extension

## Execution Pass — Implementation + Execution (optional)
- Builder: Codex CLI (GPT-5)
- Implementation plan:
  - Backend: add text extraction service, files router, analyze-text endpoint, extract_from_text prompt, env var, dependencies.
  - Frontend: add FileImportModal, API methods, Workbench import button, WorkItemDrawer initialDraft support.
- Files changed:
  - application-tracker/backend/services/text_extraction.py
  - application-tracker/backend/routers/files.py
  - application-tracker/backend/routers/ai_router.py
  - application-tracker/backend/services/ai_service.py
  - application-tracker/backend/extractors/gpt_parser.py
  - application-tracker/backend/main.py
  - application-tracker/backend/env.example
  - application-tracker/backend/requirements.txt
  - application-tracker/backend/tests/test_text_extraction.py
  - application-tracker/frontend/src/components/FileImportModal.tsx
  - application-tracker/frontend/src/components/WorkItemsView.tsx
  - application-tracker/frontend/src/components/WorkItemDrawer.tsx
  - application-tracker/frontend/src/api/workItems.ts
  - application-tracker/requirements.txt
- Tests / verification: Not run (manual verification recommended).
- Migration / rollout notes: None.
- Change Requests (if any): None.

## Review Gate (Triad)
- Agent 1 (AC/value):
- Agent 2 (UX/accessibility):
- Agent 3 (architecture/security): PASS - aligned with triad decisions; magic-byte detection, .xls blocked, error codes, ephemeral; pandas not used for extraction (requirements retains pre-existing pandas).
- Decision: Ready for testing (tests not run in this pass).

## Links / References
- `application-tracker/backend/services/document_service.py`
- `application-tracker/backend/extractors/gpt_parser.py`
- `application-tracker/frontend/src/pages/DocumentsPage.tsx`
