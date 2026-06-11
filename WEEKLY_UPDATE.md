# Weekly Update Runbook

This is the procedure for the weekly Monday refresh of the HMV Collection
Tracker. It is written to be run by Claude Code (it was previously done by
Perplexity). The whole job is: pull the latest HMV order activity from Gmail,
reconcile it into the dashboard data, recompute the headline numbers, and
commit + push.

> **Schedule:** every **Monday**. See [Scheduling](#scheduling) below for how
> the recurring session is set up.

## TL;DR prompt

> Run the weekly HMV Collection Tracker update following `WEEKLY_UPDATE.md`:
> check Gmail for new/changed HMV orders since the last update, reconcile them
> into `index.html` and `hmv_orders.json`, recompute the KPIs and the
> "Last updated" date, then commit and push.

## Step by step

### 1. Establish the cut-off

Find the date of the previous update — the most recent
`Weekly update: …` commit (`git log -1 --pretty=%s`) or the
`Last updated:` value in `index.html`. Only emails **after** that point need
to be processed.

### 2. Gather order activity from Gmail

Use the Gmail tools to search the inbox (account: redwards2k@gmail.com) for
HMV messages newer than the cut-off. Useful queries:

- `from:hmv.com newer_than:14d`
- `subject:(order OR dispatched OR delivered OR cancelled) hmv`
- `from:(hmv OR noreply@hmv.com OR orders@hmv.com)`

Read the threads and classify each into one of:

| Email type | Action on the data |
| --- | --- |
| **New order confirmation** | Add a new order object (id, date, items, prices, `total`). New items start as `Pre-order` (or `Dispatched`/`Delivered` if the email says so). |
| **Dispatch / shipping** | Set the matching item's `status` to `Dispatched`. |
| **Delivery confirmation** | Set the matching item's `status` to `Delivered`. |
| **Release-date change** | Update the item's `release_date`. |
| **Cancellation / refund** | Remove the cancelled item (and the order if it's now empty). |

Match items by `title` + `order_id`. Titles must match exactly across
`index.html`, `hmv_orders.json`, and the `itemUrls` map.

### 3. Update the data — in BOTH places

Apply every change to:

1. The inline `const orders = [ ... ]` array in `index.html`, **and**
2. `hmv_orders.json`.

They must stay byte-for-byte equivalent in content. Keep orders in the existing
order (roughly by `order_date`). For any brand-new item title, add a product
URL to the `itemUrls` map in `index.html` (use the real HMV product page if
known, otherwise an `https://www.hmv.com/search?q=<title>` fallback — match the
existing pattern).

### 4. Recompute the hardcoded headline numbers

After editing the data, recompute and update these in `index.html`:

- **Total Spent** = sum of all order `total`s (format `£X,XXX.XX`).
- **"across N orders"** sub-label = number of orders.
- **Total Items** = total number of item objects.
- **Awaiting Delivery** = count of items with status `Pre-order` or
  `Dispatched` (i.e. not yet `Delivered`).
- **Pre-order Value** = sum of `price` for items not yet `Delivered`.

Sanity-check against the JSON, e.g.:

```bash
python3 -c "import json;d=json.load(open('hmv_orders.json'));\
print('orders',len(d));\
print('items',sum(len(o['items']) for o in d));\
print('spent',round(sum(o['total'] for o in d),2));\
print('awaiting',sum(1 for o in d for i in o['items'] if i['status']!='Delivered'))"
```

### 5. Update the "Last updated" date

Set the header `Last updated: …` to **today** in `D Month YYYY` form
(e.g. `8 June 2026`).

### 6. Commit and push

Use the established commit-message style:

```
Weekly update: DD Mon YYYY - <short summary of what changed>
```

Examples from history:

```
Weekly update: 08 Jun 2026 - Dark Crystal cancelled, Monty Python delivered
Weekly update: 01 Jun 2026 - cancel/reorder Speed Racer LE, new Pink Panther, 2 delivered
```

Then push:

```bash
git push -u origin <branch>
```

### 7. If nothing changed

If there were no relevant emails since the last update, still bump the
`Last updated:` date and commit `Weekly update: DD Mon YYYY - no changes`, so it
is clear the check ran. (Optional — skip the commit entirely if you prefer a
clean history.)

## Notes & gotchas

- The dashboard renders from the **inline array**, not from `hmv_orders.json`.
  Editing only the JSON will not change the page. Always edit both.
- Items a user deletes in the UI are remembered client-side via
  `localStorage` and survive these rebuilds, so don't worry about them here.
- There is no build step or test suite — verify by opening `index.html` and
  confirming the KPIs render and the numbers look right.

## Scheduling

The recurring Monday run is a **scheduled session in Claude Code on the web**
(it cannot be created from inside a session). To set it up:

1. Open this repo in Claude Code on the web
   (https://claude.com/code) and create a **Schedule**.
2. Cadence: **weekly, Mondays**.
3. Prompt: the [TL;DR prompt](#tldr-prompt) above.
4. Make sure the environment has the **Gmail** integration connected for
   redwards2k@gmail.com so the session can read order emails.

See the docs on scheduling and triggers:
https://code.claude.com/docs/en/claude-code-on-the-web
