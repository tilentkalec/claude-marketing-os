# Data Sources

This repo doesn't ship with data-pulling scripts. You bring your own. The CMO orchestrator expects two kinds of data, dropped as JSON in `data/`:

1. **Traffic + conversions** (typically GA4)
2. **Revenue** (typically Stripe / WooCommerce / Shopify)

The contract below is what the orchestrator expects. As long as your scripts produce JSON that matches, the rest of the system works.

---

## Traffic JSON contract

Suggested filename: `data/traffic_all_markets.json`

```json
{
  "pulled_at": "2026-04-29T10:00:00Z",
  "date_range": {
    "start": "2026-04-22",
    "end": "2026-04-29"
  },
  "markets": {
    "US": {
      "market": "US",
      "rows": [
        {
          "date": "20260425",
          "source": "google",
          "medium": "organic",
          "country": "United States",
          "sessions": 1500,
          "users": 1100,
          "conversions": 32.0,
          "revenue": 480.00
        }
      ]
    },
    "UK": { "...": "..." }
  }
}
```

Required fields per row: `date`, `source`, `medium`, `sessions`, `users`, `conversions`, `revenue`.

---

## Revenue JSON contract

Suggested filename: `data/revenue_all_markets.json`

```json
{
  "pulled_at": "2026-04-29T10:00:00Z",
  "date_range": {
    "start": "2026-04-22",
    "end": "2026-04-29"
  },
  "markets": {
    "US": {
      "market": "US",
      "currency": "USD",
      "orders": 142,
      "revenue": 5840.00,
      "aov": 41.13,
      "rows": [
        {
          "order_id": "ord_...",
          "date": "2026-04-25",
          "amount": 49.00,
          "currency": "USD",
          "source": "checkout",
          "customer_id": "cus_..."
        }
      ]
    },
    "UK": { "...": "..." }
  }
}
```

Required top-level fields per market: `currency`, `orders`, `revenue`, `aov`. The `rows` array is optional but useful for cohort/source analysis.

---

## How the orchestrator uses these

The CMO reads both files at the start of report mode, slices per market, and passes relevant context to channel subagents in their briefs. For example:

> "Analyze last 7 days Klaviyo campaigns for the US market. Context: revenue is up 18% WoW and email-attributed orders made up 23% of revenue."

If a file is missing or older than the requested window, the CMO runs the user's pull scripts (paths configured in `client-config.md` or hardcoded in your scripts) and waits for fresh JSON.

---

## Caching

Files in `data/` are reused within a session if their `pulled_at` timestamp covers the requested window. To force a fresh pull, delete the file or run your pull script manually.

---

## Examples in the wild

The private companion repo (used for a real ed-tech client) has working pullers for GA4 and WooCommerce that produce JSON matching this contract. Reach out if you'd like to see reference implementations — the schema is what's canonical, not any specific implementation.
