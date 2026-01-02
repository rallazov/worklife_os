# Agent Session - Workbench Filter Truth Model - 2026-01-01
tags: #agent #triad
status: Draft

## Goal
Define a **trustworthy** Workbench filter + counts contract (tabs/areas/state/entity) so counts match the list and empty states are debuggable.

## Context Packet
- Problem / why now:
  - Workbench currently mixes “global” counts with “filtered list” results, so users can see counts that don’t match what’s on screen.
  - This is a trust issue and blocks scaling to more domains and modes.
- Links:
  - [[02-Engineering/Architecture/SPEC - Workbench Filters Contract|SPEC - Workbench Filters Contract]]
  - [[02-Engineering/Architecture/Data-Model|Data Model]]
  - [[02-Engineering/Decisions/ADR-0003-Workbench-Filter-Truth-Model|ADR-0003 Workbench Filter Truth Model]]
  - Code refs (repo): `application-tracker/frontend/src/components/WorkItemsView.tsx`
  - Code refs (repo): `application-tracker/backend/services/application_service.py`
- Constraints:
  - Every displayed count must be computed from the same base population as the list it summarizes.
  - Default view must include new items (TRIAGE) and active items (ACTIVE).
  - Filtering must remain fast enough for interactive use.
- Definition of Done:
  - Written contract for: View tabs, Area pills, State toggle, Entity filter.
  - Explicit “count invariants” + empty-state rules.
  - API shape agreed (list + counts).
  - Next actions created (backend + frontend + docs).

## Role Assignment (rotate)
- Primary: Agent 1
- Challenger: Agent 3
- Integrator: Agent 2

## Agent 1 — Product + Business + Delivery
### Notes
- User promise: “Counts are trustworthy; filters are safe; empty states explain themselves.”
- Required filter dimensions (v1):
  - View tabs: Inbox / Applications / Tasks / Learning / All
  - Area pills: domain (job_hunt, finance, parenthood, business, general, …)
  - State: Open (ACTIVE+TRIAGE) / Done / All
  - Entity: company (plus future: source)
  - Optional toggles: needs_action
  - Search: global text match
- Acceptance criteria:
  - If a pill/tab shows “(N)”, selecting it produces a list whose total equals N (modulo pagination).
  - Switching tabs does not “hide” new items by default.
  - Empty state shows active filters + 1-click safe resets.
- Success metrics:
  - Reduction in “count mismatch” confusion.
  - Faster triage completion (time-to-first-action).

### Questions / Requests
- To Agent 2: Which counts should stay global “trust bar” vs be contextual to current filters?
- To Agent 3: Can we implement a counts endpoint that shares the exact same filter logic as the list endpoint?

## Agent 2 — UX + Frontend
### Notes
- Separate **two kinds of counts** in the UI:
  1) **Sanity Bar (global trust)**: open/overdue/today/week/triage/needs_action across the whole account.
  2) **Filter counts (contextual)**: counts that match the current list’s filter base.
- Empty state rules:
  - Always render “Active filters” chips.
  - Provide ordered reset actions:
    1) Clear search
    2) Clear entity
    3) Reset state to Open
    4) Reset area to All
    5) Reset view to All
- Interaction:
  - Filter changes update URL params.
  - Debounce search; avoid re-fetch storms.
- Accessibility:
  - Tabs/pills are keyboard reachable, with visible focus.
  - State toggle has clear labels (Open/Done/All) and is screen-reader friendly.

### Questions / Requests
- To Agent 1: For “Applications” tab, should area be forced to job_hunt or still allow cross-domain apps?
- To Agent 3: For contextual counts, do we include `search` in the count base (expensive) or exclude it (but then counts won’t match)?

## Agent 3 — Architecture + Backend
### Notes
- Current state:
  - `/work-items` already supports `type, domain, state, needs_action, company, search`.
  - `/sanity-counts` is global and based on open items (`state` not in DONE/ARCHIVED).
- Proposed backend contract:
  - Keep `/sanity-counts` as-is for global trust.
  - Add `GET /work-items/counts` that uses the same filters as `/work-items` and returns grouped counts needed by the UI.
  - Counts endpoint should accept the same query params as `/work-items` (including `search`) so “N matches” remains true.
- Performance notes:
  - Group-by queries for counts are cheap relative to full list fetch, but `search` can be expensive; debounce on frontend.
  - Ensure the counts endpoint uses the same LIKE escaping as `/work-items`.

### Questions / Requests
- To Agent 1: Confirm the v1 “truth rule”: counts must match the filtered list, not just the global open population.
- To Agent 2: Which groupings do you want in v1 (`byDomain`, `byType`, `dueBuckets`, `bySource`)?

## Integration Summary (Integrator)
- We will treat “global trust counts” and “contextual filter counts” as separate products.
- Workbench’s pills/tabs must use contextual counts so numbers match what the list would show when selected.
- Backend should provide a dedicated counts endpoint that reuses the same filter logic as `/work-items` to prevent drift.

## Decisions
- Accepted:
  - Keep `/sanity-counts` as global trust metrics.
  - Add a contextual counts endpoint for Workbench filters.
  - Empty states must show active filters + safe reset actions.
- Rejected:
  - Reusing global sanity counts for Workbench pill/tab numbers (causes mismatch).

## Next Actions
- [ ] Create/update ADR: [[02-Engineering/Decisions/ADR-0003-Workbench-Filter-Truth-Model|ADR-0003 Workbench Filter Truth Model]]
- [ ] Update SPEC: [[02-Engineering/Architecture/SPEC - Workbench Filters Contract|SPEC - Workbench Filters Contract]] (add Learning tab + count rules)
- [ ] Backend: implement `GET /work-items/counts` (shared filters with `/work-items`)
- [ ] Frontend: use contextual counts for Workbench tabs/pills; keep Sanity Bar global
- [ ] UX: define empty-state copy + reset ordering

## Links / References
- [[02-Engineering/Architecture/SPEC - Workbench Filters Contract|SPEC - Workbench Filters Contract]]
- [[02-Engineering/Architecture/Data-Model|Data Model]]
- `application-tracker/frontend/src/components/WorkItemsView.tsx`
- `application-tracker/backend/services/application_service.py`

## Handoff Packet (for Builder)
- Scope/contract links (SPEC/ADR):
  - [[02-Engineering/Decisions/ADR-0003-Workbench-Filter-Truth-Model|ADR-0003 Workbench Filter Truth Model]]
  - [[02-Engineering/Architecture/SPEC - Workbench Filters Contract|SPEC - Workbench Filters Contract]]
- Acceptance criteria:
  - Tab/pill counts match the list results they summarize.
  - Sanity Bar remains global; Workbench filter counts are contextual.
  - Empty state shows active filters + safe reset actions.
- Non-goals:
  - Re-architecting sync/extraction.
  - Precomputing counts asynchronously (future).
- Constraints (security/privacy/perf):
  - Avoid storing new sensitive data.
  - Debounce search; `search` included in contextual counts to preserve truth.

## Execution Pass — Implementation + Execution (optional)
- Builder:
- Implementation plan:
- Files changed:
- Tests / verification:
- Migration / rollout notes:
- Change Requests (if any):

## Review Gate (Triad)
- Agent 1 (AC/value):
- Agent 2 (UX/accessibility):
- Agent 3 (architecture/security):
- Decision:

