# Ontario Multifamily Market Intelligence

Personal deal-sourcing and underwriting dashboard for Ontario multifamily (5–15 unit) acquisitions, built around the 3–4 year capital recovery model (D'Souza methodology).

**Live app (after enabling GitHub Pages):** `https://<your-username>.github.io/ontario-market-intel/`

## What it does

- **Market explorer** — 14 Ontario regions scored on cap rate, vacancy, rent growth, population and employment growth, with price-per-unit and 1BR/2BR market rents. Table and card views, sortable and filterable.
- **Live listings by area** — current multifamily listings (LoopNet / broker public data) with a system-fit assessment against the capital recovery criteria. Refresh button pulls the latest `data/*.json`.
- **Underwriting popup** — click "Underwrite" on any listing. Fully editable analyzer replicating the Property Analyzer spreadsheet: Ontario land transfer tax formula, staying power fund, 25/30/40-year amortization scenarios with Canadian semi-annual compounding (verified to the cent against the source sheet), DCR from both your view and the lender's view, ROI stack (cash + paydown + appreciation), and the capital recovery gate — the pass/fail filter question.
- **Persistence** — target markets, listing watchlist, per-deal underwriting, and field notes all save to localStorage.

## Architecture

```
index.html          the whole app (no build step, no dependencies)
data/
  markets.json      regional benchmarks  { vintage, markets: [...] }
  listings.json     current listings     { vintage, listings: [...] }
```

The app ships with embedded fallback data so `index.html` also works opened directly from disk. When hosted (GitHub Pages, any static server, or your homelab), the **↻ Refresh data** button fetches `data/*.json` with cache busting — so updating the data is just committing new JSON.

## Deploy to GitHub Pages

```bash
cd ontario-market-intel
git init
git add .
git commit -m "Ontario multifamily market intelligence dashboard"
git branch -M main
git remote add origin git@github.com:<your-username>/ontario-market-intel.git
git push -u origin main
```

Then: repo **Settings → Pages → Source: Deploy from a branch → main / (root) → Save.**
The app is live a minute later at `https://<your-username>.github.io/ontario-market-intel/`.

## Refreshing the data

Listings sites (LoopNet, Realtor.ca) block browser cross-origin requests and bots, so the refresh is a two-step loop:

1. Ask Claude: *"Refresh my ontario-market-intel listings — sweep LoopNet for 5–15 unit multifamily in my target areas."* Claude returns updated `listings.json` (and `markets.json` when benchmarks move).
2. Commit and push. Everyone viewing the Pages URL gets the new data on next refresh click.

To automate later: a small FastAPI service (or GitHub Action on a schedule) that scrapes via a proxy service and commits the JSON. The dashboard needs no changes — it just reads the JSON.

## Data schema

`data/listings.json`:

```json
{
  "vintage": "Fetched July 27, 2026",
  "listings": [
    {
      "area": "Oshawa / Durham",
      "name": "300 Athol St, Whitby",
      "price": 2100000,
      "units": 8,
      "unitMix": "8 × 2BR",
      "sf": 8202,
      "broker": "Royal LePage Frank",
      "listed": "Jun 2026",
      "source": "LoopNet",
      "url": "https://...",
      "fit": "strong | potential | weak",
      "fitNote": "One-paragraph assessment against the capital recovery system"
    }
  ]
}
```

`price` and `units` may be `null` (price on request). `fit` drives the badge and filter.

## Methodology notes

- Mortgage payments use Canadian semi-annual compounding: `r = (1 + rate/2)^(1/6) − 1`.
- Land transfer tax uses the Ontario (non-Toronto) bracket formula.
- Staying power fund defaults to one month's total income (overridable).
- Lender's-view DCR applies floors: vacancy ≥ 3%, property management ≥ 5%, repairs ≥ $750/unit/yr.
- Capital recovery gate: pass = a 75% LTV refinance at the area cap rate returns ≥ 90% of invested capital within 4 years at the assumed NOI growth.
- Kingston's headline rent growth is supply-driven, not organic demand — its score is penalized deliberately.

Data sources: CMHC, CBRE, StatCan, LoopNet, broker listings. All figures are point-in-time snapshots — verify against actuals (rent rolls, 12–24 months of utility bills, tax bills) before making offers.
