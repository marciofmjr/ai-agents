---
name: zzzteste
description: Lightweight test skill for validating skill triggering and execution flow. Use when the user explicitly mentions "zzzteste" or asks to run a basic skill smoke test.
---

# zzzteste

Execute a minimal smoke-test workflow.

## Workflow

1. Confirm the user asked to run `zzzteste`.
2. Reply with `zzzteste:ok`.
3. Keep the response short unless the user asks for details.

## Notes

- Treat this skill as a trigger and pipeline sanity check.
- Do not perform unrelated file edits when this skill is invoked.
