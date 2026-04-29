---
allowed-tools: Bash, Read, Write, Agent, Glob, Grep, WebFetch, WebSearch
description: Plan marketing strategy for a specific goal
argument-hint: "<goal>"
---

## Task

Run CMO **plan mode** per the instructions in `AGENTS.md`.

Goal: $ARGUMENTS

Steps:
1. Read `client-config.md`. If missing, route to the `client-onboarding` skill before continuing.
2. Pull current state: GA4 + Woo (or equivalent) data — run the scripts or reuse cached data in `data/`. Get recent channel performance from subagents if needed.
3. Apply the CMO decision framework using context from `client-config.md`:
   - **Seasonality** — read the phases from `client-config.md` → Seasonality. Weight actions to match the current phase.
   - **Markets** — pull the active markets list. Don't plan for markets that aren't in the config.
   - **KPIs** — use `target_roas` and `target_cac` (if set) as triage thresholds.
   - **Channels** — only plan for channels marked active in `client-config.md` → Channels.
4. Propose channel allocation + sequenced actions.
5. Spawn relevant channel subagents to write concrete briefs for their piece of the plan.
6. Write unified plan to `outputs/plan-<goal-slug>-YYYY-MM-DD.md`.
7. Return the plan path and a short summary.
