# Bella Goose Coffee — Business Intelligence Project

End-to-end data analysis and BI dashboard project for a three-location specialty coffee company operating in a Mountain-timezone tourist town. Built from raw Square POS exports (orders, payments, labor, catalog) and delivered as self-contained interactive HTML dashboards.

This repo contains the full pipeline — from raw CSV → pandas transformations → JSON datasets → interactive dashboards — with the complete Jupyter notebook documenting every step for the flagship location's 3-year deep dive.

---

## What's in here

| File | Description |
|---|---|
| `bella_goose_river_3yr_deep_dive.html` | **Flagship deliverable.** 8-tab interactive dashboard covering 3 years of the main location (Cafe on the River). 30+ charts, sortable product tables, revenue heatmap, P&L, and a 9-point action plan. |
| `bella_goose_tourist_report.html` | Single-year snapshot of Location 1 (Main). Tourist-season framing, key KPIs, actionable recommendations. |
| `bella_goose_downtown_report.html` | Location 2 (Downtown). Seasonal coffee shop with different traffic patterns from the main location. |
| `bella_goose_briggsville_report.html` | Location 3 (Briggsville). Production hub + seasonal retail. Includes labor analysis showing most cost is production (baking, syrup, roasting) supporting all three sites. |
| `bella_goose_data_pipeline.ipynb` | **Complete pandas notebook** for the 3-year deep dive. 21 sections, 37 code cells. Loads raw Square CSVs, converts timezones, fixes the critical order-level deduplication bug, builds all 29 analytical datasets, and exports to JSON. |

---

## The project

### Business context

Bella Goose Coffee operates three sites in a small tourist town: a flagship cafe on the river, a downtown location, and a production hub in Briggsville that bakes, roasts, and makes syrups for all three while also running a seasonal retail storefront.

The analysis answers questions the owner actually cares about:

- Is the business getting less tourist-dependent over time, or are we still at the mercy of summer?
- Which products are pulling their weight, and which discontinuations cost us real revenue?
- How efficient is our labor, and where does the money actually go?
- What are the highest-ROI moves we could make in the next 90 days?

### Data

Four CSV exports from Square POS covering March 2023 through March 2026:

- `square_orders.csv` — 728K rows, 291K unique orders. **One row per line item**, not per order (see caveat below).
- `square_payments.csv` — 268K payment records with processing fees, tips, card brands.
- `square_labor.csv` — 16K shift records with hourly rates and job titles.
- `square_catalog.csv` — 1,165 menu items.

### Stack

- **Python / pandas** for all data transformation
- **Jupyter** for the documented pipeline notebook
- **Chart.js 4.4.1** for dashboard visualizations
- **Vanilla JS + HTML + CSS** for the self-contained interactive reports (no build step, no backend, just open the file in a browser)

---

## The critical bug the notebook catches

The Square orders CSV has **one row per item, not per order**. Order-level fields (`order_total`, `order_discount`, `order_tip`) are duplicated across every item row within a single order. Naively summing them massively overcounts.

The original 3-year analysis reported $538K in discounts on $2.12M revenue — a 25% rate that would be absurd for a coffee shop. The real number was **$80.9K (3.7%)**, a totally healthy loyalty program spend.

The fix is to deduplicate to order level before summing order-level fields:

```python
order_level = orders.groupby('order_id').agg(
    order_discount=('order_discount', 'first'),
    order_tip=('order_tip', 'first'),
    order_total=('order_total', 'first'),
    net_sales=('net_sales', 'sum'),        # item-level, safe to sum
    item_discount=('discount', 'sum'),     # item-level, safe to sum
).reset_index()
```

The notebook demonstrates the bug explicitly, shows the 6.3× overcount on discounts and 4.3× on tips, and cross-checks the corrected tip total against the payments table (which has no duplication issue) — matching within $7 across 291K orders.

---

## Key findings from the 3-year deep dive

- **Revenue nearly doubled in 3 years**: $1.22M (2023) → $1.77M (2024) → $2.12M (2025). 31.7% CAGR.
- **Less tourist-dependent every year**: off-season revenue share grew from 26% of total (2023) to 37% (2025). The floor is rising while peaks stay strong.
- **Weekends carry the week**: Saturday and Sunday do $7.4K and $7.6K/day respectively, vs ~$4K on weekdays.
- **Peak slot**: 8–9 AM on weekends in peak season. One hour on one day is worth staffing exactly right.
- **Food attach rate is declining**: 63.4% → 56.9% over 3 years. Each percentage point is roughly $12–21K in annual food revenue.
- **Egg Sammie revenue cliff**: $139K (2024) → $14K (2025). The discontinuation wasn't fully replaced by the cheddar variants.
- **Labor is efficient and improving**: revenue per labor hour grew 15% from $50 to $58 while total labor held at ~23% of revenue.
- **Discounts are healthy**: 3.7% of gross sales, down from 4.1%.

---

## Running the notebook

```bash
pip install pandas numpy nbformat

# Point DATA_DIR in the notebook at wherever your Square CSVs live
jupyter notebook bella_goose_data_pipeline.ipynb
```

The notebook is self-contained and produces a single JSON file (`dash_main_corrected.json`) that feeds the dashboard builder. Every code cell runs clean against the source data and prints a verifiable output so you can sanity-check each step.

---

## Viewing the dashboards

Just open any of the `.html` files directly in a browser. They're fully self-contained — data is embedded as JSON, Chart.js loads from CDN, no server required. Works offline after first load. Tested in Chrome, Safari, and Firefox.

---

## Notes on methodology

- **All times are US/Mountain** (not UTC). Critical for hourly heatmaps and day-of-week analysis in a Mountain-timezone business.
- **Season definition**: Peak = Jun–Aug, Shoulder = May, Sep, Oct, Off-Season = Nov–Apr. Chosen to match actual tourist traffic patterns, not calendar quarters.
- **Drink vs Food classification** is keyword-based on item names. Customize the keyword list for your own menu if forking.
- **Cancellation data was excluded** from the final dashboard after investigation showed 95% of "cancellations" were Kyoo mobile-app cart abandonments ($0 orders), not real revenue loss. The pipeline still computes the metric for reference, but the dashboard reframes around signals that are actually actionable.

---

## License & attribution

Built by Zachari as a portfolio project. Raw POS data is proprietary and not included — the notebook expects you to supply your own Square exports with matching schemas.
