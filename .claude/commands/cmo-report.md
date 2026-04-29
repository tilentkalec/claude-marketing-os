---
allowed-tools: Bash, Read, Write, Agent, Glob, Grep
description: Run a cross-channel CMO performance report
argument-hint: [weekly|monthly]
---

## Task

Run CMO **report mode** per the instructions in `AGENTS.md`.

Argument: $ARGUMENTS (defaults to "weekly" if empty)

Steps:
1. Read `client-config.md`. If missing, route to the `client-onboarding` skill before continuing.
2. Map argument to days: `weekly` → 7, `monthly` → 30. If empty, use 7.
3. Check `data/ga4_all_markets.json` and `data/woo_all_markets.json`. If both exist and their `pulled_at` timestamp covers the requested date range (or was pulled within the current session), reuse them. Otherwise run the relevant pull scripts (paths and credentials per the user's setup; see README).
4. Read the resulting JSON files from `data/`.
5. Spawn the channel subagents listed in `client-config.md` → Channels (active ones only) in parallel via the Agent tool. Each brief must include date range, markets (read from `client-config.md`), and any cross-channel context you see in the GA4/Woo data.
6. Synthesize into the report structure defined in `AGENTS.md` (Report mode).
7. Write to `outputs/cmo-report-YYYY-MM-DD.md`.
8. Return the report path and a short summary to the user.
