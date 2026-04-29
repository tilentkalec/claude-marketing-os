# Email Super-Agent Design Spec

**Status:** Implemented as `skills/email-klaviyo.md`
**Goal:** Build a comprehensive email marketing agent skill that serves as the operational backbone for any Klaviyo-powered brand — single market or multi-market.

## Context

A typical email marketing skill is analysis/reporting only. This spec defines a **full-lifecycle** email marketing agent covering: strategy, campaign writing, flow building, performance analysis, optimization, and reporting — all in one skill.

## Architecture

**Single skill file** (`skills/email-klaviyo.md`) with modular sections you can invoke independently:

### Module 1: Strategy
- Input: client brief (product, ICP, current metrics, goals, seasonality from `client-config.md`)
- Output: full email strategy doc — flows to build, segments to create, campaign cadence, KPI targets
- References industry benchmarks (trial conversion, churn, flow performance)

### Module 2: Campaign Writing
- Input: campaign goal, audience segment, brand voice (from config), past top-performers
- Output: 3-5 subject lines, preview text, full email body, CTA
- Pulls past performance from the Klaviyo REST API to inform decisions
- For multi-market brands: writes English first, then translates + localizes per market using the localization rule in `client-config.md` → Markets

### Module 3: Flow Building
- Input: flow type (welcome, trial-to-paid, churn prevention, feature adoption, failed payment, upsell)
- Output: complete multi-email sequence — email count, timing, triggers, branch logic, full copy per email
- SaaS / B2C lifecycle best practices baked in

### Module 4: Performance Analysis
- Pulls Klaviyo campaign + flow data via REST API (one key per market)
- Computes: revenue attribution, conversion rates, engagement trends
- Flags underperformers with specific fix recommendations
- Generates an operator-ready report

### Module 5: Optimization
- A/B test recommendations from past data
- Segment refinement suggestions
- Send time optimization
- Deliverability monitoring

### Module 6: Reporting
- Monthly performance report with revenue attribution
- Email-attributed MRR, trial conversion, churn metrics
- Benchmarks vs. industry averages
- Operator-ready format

## Key Design Decisions

- **Klaviyo-first:** All API access targets Klaviyo. Adapting to Mailchimp / Customer.io would mean adding a parallel skill, not modifying this one.
- **Modular:** Each module can be invoked independently ("write a campaign" vs. "full strategy").
- **Multi-market by default:** Markets, localization rules, and per-market account keys all read from `client-config.md` and `.env`.
- **REST API, not MCP:** The Klaviyo MCP only scopes to one account at a time, which breaks for any multi-market brand. The skill always calls the REST API directly.
- **`client-config.md`-driven:** Brand context, KPIs, seasonality, and market list are never hardcoded — every run starts by reading the config.
