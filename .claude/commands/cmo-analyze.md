---
allowed-tools: Bash, Read, Write, Agent, Glob, Grep, WebFetch, WebSearch
description: Ask the CMO a targeted cross-channel question
argument-hint: "<question>"
---

## Task

Run CMO **analyze mode** per the instructions in `AGENTS.md`.

Question: $ARGUMENTS

Steps:
1. Read `client-config.md`. If missing, route to the `client-onboarding` skill before continuing.
2. Interpret the question. Decide which subagents are relevant — not always all of them. Match the question to the channels in `client-config.md` → Channels.
3. Pull only the GA4/Woo (or equivalent) slices needed for context, or reuse cached data in `data/` if fresh.
4. Brief the selected subagents with the specific question + relevant context via the Agent tool.
5. Synthesize a focused inline answer. Do not write to a file unless the user asks.
