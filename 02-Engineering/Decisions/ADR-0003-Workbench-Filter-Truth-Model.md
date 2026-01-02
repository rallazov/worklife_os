# ADR-0003 - Workbench Filter Truth Model
tags: #adr
status: Proposed
date: 2026-01-01

## Context
Workbench must provide **trustworthy** counts and safe filtering.

Today, the UI can show counts that come from a different population than the filtered list (e.g., global open-item counts shown next to filter controls). This causes user confusion (“why does it say 12 but I see 3?”) and blocks scaling to more domains.

## Decision
1) Define filter dimensions (v1)
- View: `inbox | applications | tasks | learning | all`
- Area: domain (job_hunt, finance, parenthood, business, general, …)
- State: Open (ACTIVE+TRIAGE) | Done | All
- Entity: company (future: source)
- Optional: needs_action
- Search: text match

2) Split counts into two categories
- **Global sanity counts** (`/sanity-counts`): always computed over all open items for the user (state not in DONE/ARCHIVED), independent of the current Workbench filter context.
- **Contextual filter counts** (new): computed from the same base query as the Workbench list, so counts match what the list would show when a filter is selected.

3) Add `GET /work-items/counts`
- Accept the same filter params as `GET /work-items`.
- Return grouped counts needed by the Workbench UI (v1 likely `byDomain`, `byType`, and due buckets).
- Implement via shared query builder / shared filter function to prevent drift.

## Alternatives Considered
- Use global sanity counts everywhere (simpler, but misleading for filtered lists).
- Client-side counts from loaded items (incorrect with pagination).
- Precompute counts asynchronously (faster at scale, but adds complexity; consider later).

## Consequences
- Backend: implement and maintain a counts endpoint with guaranteed parity to list filters.
- Frontend: render contextual counts for pills/tabs; keep sanity bar as a separate trust widget.
- Product: reduced confusion and stronger trust (“numbers match what I see”).

## References
- Session: [[04-Execution/Agent-Sessions/Agent Session - Workbench Filter Truth Model - 2026-01-01|Agent Session - Workbench Filter Truth Model - 2026-01-01]]
- SPEC: [[02-Engineering/Architecture/SPEC - Workbench Filters Contract|SPEC - Workbench Filters Contract]]
- Code: `application-tracker/backend/services/application_service.py`
- Code: `application-tracker/frontend/src/components/WorkItemsView.tsx`
