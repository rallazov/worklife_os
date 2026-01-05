# Agent Session — WorkItemsView Refactor Plan
tags: #agent #refactor
status: Draft

## Goal
- Reduce complexity and tech debt in WorkItemsView without behavior regressions.
- Prepare the code for UX and accessibility improvements.

## Scope / Guardrails
- Only update `application-tracker/frontend/src/components/WorkItemsView.tsx` and co-located helper components/hooks if needed.
- Keep hybrid styling (MUI `sx` + className) unless a clear best-practice issue forces change.
- No backend changes, no new dependencies, no feature expansion.
- Preserve current behavior, URL-driven filters, and keyboard navigation.

## Decisions
- Accepted:
  - Refactor scope limited to WorkItemsView and co-located helpers.
  - Keep hybrid styling for now.
  - Iterative refactor with small, reversible steps.
- Rejected:
  - Full styling unification (MUI-only or Tailwind-only).
  - Cross-module refactors outside WorkItemsView.

## Refactor Plan (Iterative)
1. Guardrails + baseline: define manual checklist; extract magic numbers to constants.
2. Structural split: extract header/filters/sanity/presets/table/empty state/bulk bar into local components (same file or co-located).
3. State consolidation: add helper for URLSearchParams updates; centralize error handling.
4. UX + a11y pass: add aria-pressed, header scope, aria-live for counts; improve mobile table overflow; add loading skeleton.
5. Tests/verification: add unit tests for filter parsing + shouldInsertNewItem; integration test for quick actions.

## Verification
- Manual: filters, quick actions, bulk actions, import modal, keyboard navigation.
- Automated: frontend unit tests for filter helpers and insertion logic.

## Links / References
- Code: `application-tracker/frontend/src/components/WorkItemsView.tsx`
