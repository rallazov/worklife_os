# Triad Workflow (3-Agent Collaboration)
tags: #agent #workflow
status: Draft

## Purpose
Use 3 agents to reduce blind spots while keeping context small.

## The 3 agents
- Agent 1: Product + Business + Delivery
- Agent 2: UX + Frontend (Experience)
- Agent 3: Architecture + Backend (System)

## Session roles (rotate every session)
- Primary: drafts the first proposal
- Challenger: critiques assumptions, edge cases, alternatives
- Integrator: synthesizes a final decision + action plan

## Where outputs go
- Long-term decision: `02-Engineering/Decisions/` (ADR)
- Behavior/contract: `01-Product/` or `02-Engineering/Architecture/` (SPEC)
- Operational process: `03-Ops/Runbooks/` (RUNBOOK)
- Work planning: `04-Execution/Backlog.md` + `04-Execution/Sprint.md`

## Prompts
- Prompt library: [[04-Execution/Agents/Prompt-Library|Prompt Library (Triad)]]

## How to run a session (client intake default)
1) Create a new note from `04-Execution/Agent-Sessions/Agent-Session-Triad-Template.md`.
2) Client fills only “Client Ask” (3–10 lines).
3) Agent 1 completes “Agent 1 Intake” and recommends: ✅ Accept / ❌ Decline / ❓ Need more info.
4) Agent 2 reviews the intake for UX/front-end implications.
5) Agent 3 reviews the intake for architecture/back-end/security implications.
6) Integrator writes the final decision + rationale + next actions.
7) If accepted, convert durable outcomes into ADR/SPEC/RUNBOOK and link them from the session note.

## Guardrails (to prevent drift)
- Always link the source docs you relied on.
- Prefer small, testable decisions over big rewrites.
- If there is disagreement, capture it in `Alternatives Considered` (ADR).
- If declining, state the reason and (when possible) propose a safer alternative.

## Execution Pass (optional)
After the Triad locks decisions:
1) Populate the session note’s “Handoff Packet (for Builder)”.
2) A Builder agent (Claude Code / Gemini / etc.) implements against the linked SPEC/ADR and writes back under “Execution Pass — Implementation + Execution”.
3) If implementation requires changing scope/contract, the Builder files a “Change Request” and stops until the Triad resolves it.

## Review Gate (required for shipping)
- Agent 1 verifies acceptance criteria + business risk.
- Agent 2 verifies UX flows/states + accessibility.
- Agent 3 verifies architecture/security/perf.
