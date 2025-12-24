# Sync Engine
tags: #spec

## Goals
- Concurrency-limited parallel sync
- Per-user locking + stall recovery
- Timeouts to prevent hangs
- Idempotent inserts + dedupe

## Guardrails
- Never run infinite jobs
- Always record sync_started_at, sync_heartbeat, status
