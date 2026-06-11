# HMV Collection Tracker

A personal dashboard tracking HMV 4K UHD Blu-ray and SteelBook purchases for
Robin Edwards.

- **Live site:** https://hmv.starbix.uk (GitHub Pages, see `CNAME`)
- **Repo:** https://github.com/rendez2k/hmv-collection-tracker
- **Maintained with:** Claude Code (weekly automated updates — see
  [`WEEKLY_UPDATE.md`](WEEKLY_UPDATE.md))

## What it is

A single static page (`index.html`) that renders a dashboard from order data:
KPI cards (total spent, item/order counts, awaiting delivery, pre-order value),
charts (Chart.js), and tables of recent orders, upcoming releases, and full
order history. No build step and no backend — just open `index.html`.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The entire dashboard: markup, styles, and the order data. |
| `hmv_orders.json` | Machine-readable mirror of the order data. |
| `CNAME` | Custom domain for GitHub Pages (`hmv.starbix.uk`). |
| `WEEKLY_UPDATE.md` | Runbook for the weekly Monday data update. |

## Data model

Order data lives in **two** places that must be kept in sync:

1. The inline `const orders = [ ... ]` array near the bottom of `index.html`
   — this is what the page actually renders.
2. `hmv_orders.json` — a standalone JSON mirror of the same data.

Each order has this shape:

```json
{
  "order_id": "11825469",
  "order_date": "2025-09-05",
  "items": [
    {
      "title": "Tron Limited Edition 4K Ultra HD SteelBook",
      "price": 34.99,
      "release_date": "2025-09-29",
      "status": "Delivered"
    }
  ],
  "total": 144.96
}
```

- `release_date` is `null` when not yet announced (typically `Pre-order` items).
- `status` is one of: `Pre-order`, `Dispatched`, `Delivered`. Cancelled items
  are removed entirely.

In addition to the order array, these values are **hardcoded** in `index.html`
and must be recomputed on every update:

- The four KPI cards: **Total Spent**, **Total Items**, **Awaiting Delivery**,
  and **Pre-order Value** (plus the "across N orders" sub-label).
- The **`Last updated: …`** date in the header.
- Each item's HMV product URL lives in the `itemUrls` map; add an entry for any
  new title so it links correctly.

## How updates work

Once a week (Monday) the tracker is refreshed from HMV order/dispatch/delivery
emails in Gmail. The full step-by-step process — and the prompt to run a
scheduled session against — is in [`WEEKLY_UPDATE.md`](WEEKLY_UPDATE.md).
