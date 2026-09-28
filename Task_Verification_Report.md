# Final Task Verification Report
## Snapdeal Product Analytics — Internship Project

This document verifies each of the six assigned tasks against the original assignment requirements, confirms what was legitimately built, and flags every limitation with its cause. Use this as the final pre-submission checklist and as supporting evidence in the Project Report.

---

## Dataset Foundation (applies to all tasks)

| Item | Value |
|---|---|
| Raw scraped rows | 5,660 |
| Unique products (analytical base, after Decision A/B) | 4,922 |
| Categories (Top Section) | 5: Accessories, Footwear, Kids' Fashion, Men's Clothing, Women's Clothing |
| Scrape window | ~5 hours, 2 calendar dates (Aug 15–16, 2025) — single session, not historical |
| Fields confirmed genuinely absent (not proxyable) | Sales, Stock Value/Quantity, Return Rate, Promotion Flag, Seller |

**Key modeling decisions (all evidence-based, none arbitrary):**
- Decision A: `Rating (listing)` discarded (proven = −Review Count in 100% of rows); `Rating` = `Rating (detail)`; `Review Count` = `Reviews Count (listing)`; zero-rating rows (1,274, exact 100% overlap with zero-review) treated as "unrated," not "1-star" — rated-only filter (`Review Count > 0`) applied consistently wherever a rating benchmark is used.
- Decision B: 738 duplicate rows removed within category (proven identical Price/Rating across duplicates); 4 cross-category "baby clothing" spillover rows reassigned to Kids' Fashion based on matching Subcategory label and product content, not arbitrary choice.
- Decision C: 15 missing-discount rows set to 0% — validated exactly, all 15 have Price = Original Price.

---

## Task 1 — Pricing vs Satisfaction Exception Detection

| Requirement | Status |
|---|---|
| Price > category average price | ✅ Built — `ALLEXCEPT`-based measure, verified row-context-safe |
| Rating < category average rating | ✅ Built — rated-only average, excludes zero-review distortion |
| Risk Score in top 20% | ❌ Cannot compute — requires Stock Value (absent) |
| Sales < category median sales | ❌ Cannot compute — requires Sales (absent) |
| Table using only measures | ✅ Confirmed — no calculated column used; flag evaluated via `SELECTEDVALUE` + measure-to-measure calls, applied as an Advanced visual-level filter |

**Result:** 664 of 4,922 products (13.5%) flagged on the 2 supported conditions — independently verified against Python recomputation, exact match.

**Verdict: 2 of 4 conditions fully supported and correctly implemented. Risk Score and Sales-median conditions genuinely blocked by missing fields — documented, not fabricated.**

---

## Task 2 — Promotion Effectiveness Trend Analysis

| Requirement | Status |
|---|---|
| 30-day non-promotional baseline | ❌ Cannot compute — only 2 calendar dates exist |
| Promotional vs. non-promotional distinction | ❌ Does not exist — no flag field, and "high discount = promo" assumption explicitly avoided |
| Promotional lift | ❌ Depends on the above — cannot compute |
| Daily/time-based discount chart | ⚠️ Built as an honest substitute — hourly discount variation across the single scrape window, correctly chronologically ordered (19→0), straight (non-smoothed) lines |

**Verdict: Genuinely blocked at the structural level — confirmed against `snapdeal.py` (single-run script, no scheduling). Honest substitute built and clearly disclosed; full DAX template documented for future use with real historical data.**

---

## Task 3 — Discount vs Rating Causation Analysis

| Requirement | Status |
|---|---|
| Scatter: Discount % (X), Rating (Y), Review Count (Size), Product (Details), Category (Legend) | ✅ Built — one bubble per unique product (via Product URL), category available via slicer |
| Trend/regression line | ⚠️ Native visual trend line unavailable in this chart configuration — replaced with a calculated Pearson correlation coefficient measure, independently verified |
| Discount vs review volume | ✅ Built |
| Rating vs review volume | ✅ Built |
| Rating across review-count segments | ✅ Built — Low/Medium/High/No-Reviews bands, correctly excludes zero-rating distortion |
| Correlation ≠ causation separation | ✅ Explicit disclosure text distinguishes observed relationship / interpretation / possible explanation / causal conclusion |
| Sales-based causation | ❌ Explicitly stated as untestable — no Sales field |

**Result:** Correlation (Discount % vs Rating, rated-only, n=3,814) = **−0.07** — independently verified, negligible linear relationship. Review-count segments show a mild inverse pattern (~4.1 low-review vs ~4.0 high-review), explicitly flagged as a possible selection-effect, not a confirmed cause.

**Verdict: Fully supported and complete, with an honest workaround for the missing native trend line.**

---

## Task 4 — Financially Risky Inventory KPI

| Requirement | Status |
|---|---|
| Risk Score = Discount % × (1−Rating/5) × Stock Value | ❌ Cannot compute — Stock Value absent |
| Financially At-Risk Inventory % | ❌ Cannot compute — depends entirely on Stock Value |
| KPI banding (Green/Yellow/Red) | ❌ Not applicable — no real percentage to band |

**Important correction made during build:** an initial version substituted Price for Stock Value (explicitly prohibited by the original rules) — this was caught, fully removed, and replaced with a correctly-scoped supplementary chart (`Discount-Adjusted Rating Gap`, using only Discount % and Rating, never labeled as financial/inventory) plus a full disclosure and non-functional DAX template.

**Verdict: Cannot be completed — the most severely blocked task, since the entire KPI structure depends on one missing field. Correctly documented rather than faked; full DAX template ready for when Stock Value becomes available.**

---

## Task 5 — Trust-Weighted Rating Index

| Requirement | Status |
|---|---|
| Trust Score = Rating × ln(1+Review Count) × (1−Return Rate) | ❌ Cannot compute fully — Return Rate absent |
| Simple Average Rating | ✅ Built (rated-only, 4.05) |
| Review-Weighted Rating | ✅ Built (3.99) |
| Trust-Weighted Average Rating | ⚠️ Partial — "Adjusted Trust Score (No Return Rate)" = Rating × ln(1+Review Count), explicitly labeled as excluding the Return Rate term, never presented as the full Trust Score |
| Return Rate = 0% assumption | ❌ Explicitly avoided, per mentor instruction |

**Result (rated-only, natural log):** Simple 4.05, Weighted 3.99, Adjusted Trust Score 13.10 — independently verified exact match. Category breakdown shows Footwear (high review volume, median 48) scoring notably higher (15.95) than Accessories (low review volume, median 6, score 9.48) despite similar raw ratings — a genuine, data-backed illustration of why trust-weighting matters.

**Verdict: 2 of 3 metrics fully supported; third is an explicitly partial, clearly-labeled substitute. Return Rate limitation documented, not assumed away.**

---

## Task 6 — Context-Aware Price Banding

| Requirement | Status |
|---|---|
| P30 / P70 dynamic percentiles | ✅ Built with `PERCENTILEX.INC` + `ALLSELECTED` |
| Low/Medium/High classification | ✅ Built entirely as measures (disconnected "Price Band Table" + `SELECTEDVALUE` + `SWITCH` pattern) — no calculated column used |
| Responds to Category/Brand/Date slicers | ✅ Verified — Footwear-filtered test matched the predicted dynamic recalculation (Low 223/Mid 304/High 220 predicted vs. observed ~215/325/207), and clearly diverged from the static/broken-scenario prediction (Low 226/Mid 185/High 336) |
| Explanation: why calculated column fails | ✅ Documented — static, refresh-time-only evaluation vs. measures' per-context recalculation |

**Verdict: Fully supported, fully correct, and independently verified with real dynamic-behavior proof — not just a claim.**

---

## Summary Table

| Task | Fully Supported | Partially Supported | Blocked (missing field) |
|---|---|---|---|
| 1 — Exception Detection | Price & Rating conditions | — | Risk Score, Sales-median |
| 2 — Promotion Trend | — | Hourly substitute chart | 30-day baseline, promo flag, lift |
| 3 — Discount/Rating Causation | Full scatter + supporting analysis | Trend line → correlation workaround | Sales-based causation |
| 4 — Inventory Risk KPI | — | — | Entire task (Stock Value) |
| 5 — Trust-Weighted Rating | Simple & Weighted rating | Adjusted Trust Score (no Return Rate) | Full Trust Score (Return Rate) |
| 6 — Dynamic Price Banding | Fully complete | — | — |

**Missing fields responsible for every limitation above:** Sales, Stock Value, Return Rate, Promotion Flag — none exist in `snapdeal_products.csv`, and none exist in `snapdeal.py`'s collection logic. None were invented, estimated, or proxied at any point in this project.

---

## Final Integrity Confirmation

- [x] No Sales value fabricated anywhere
- [x] No Stock Value fabricated anywhere (one incorrect attempt was made, caught, and fully removed)
- [x] No Return Rate assumed to be 0%
- [x] No promotional dates arbitrarily invented
- [x] No missing historical dates manufactured
- [x] Every DAX measure uses real column names from the cleaned model
- [x] Calculated columns vs. measures correctly distinguished throughout (justified case-by-case)
- [x] Filter context (`ALLEXCEPT` vs `ALLSELECTED`) correctly matched to each task's actual requirement, not used interchangeably
- [x] Correlation never described as causation
- [x] Every business observation traced to an independently-verified real number
- [x] GitHub repo used only for hosting, never as a data source
