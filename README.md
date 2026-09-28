# Snapdeal Product Analytics — Internship Project

Power BI dashboard analyzing Snapdeal product listing data: pricing vs. satisfaction exceptions, discount variation, rating trust-weighting, and dynamic price banding.

**Live dashboard:** [PASTE PUBLISH-TO-WEB LINK HERE]

## Contents
- `snapdeal_products.csv` — raw scraped dataset (5,660 rows, single ~5-hour scraping session)
- `snapdeal.py` — Selenium-based scraper source code
- `Snapdeal_Dashboard.pbix` — Power BI report file
- `Task_Verification_Report.md` — each task checked against the assignment requirements
- `Project_Report.md` — full internship report
- `screenshots/` — one image per dashboard page

## Data Preparation Summary
- 5,660 raw rows reduced to 4,922 unique products after removing duplicate re-captures within each category
- `Rating (listing)` excluded (a scraper parsing defect: it equals the negative of the review count in every row); `Rating (detail)` used instead
- 1,274 unrated listings (zero reviews) kept in the data but excluded from rating benchmarks

## Task Status
| Task | Status |
|---|---|
| 1. Pricing vs satisfaction exceptions | Partial: price and rating conditions built (664 products); Risk Score and Sales conditions not computable |
| 2. Promotion effectiveness | Partial: hourly discount variation only; no promotion flag or 30-day history exists |
| 3. Discount vs rating | Complete (correlation −0.07); Sales-based causation untestable |
| 4. Financially risky inventory | Not computable: no Stock Value field. DAX template documented |
| 5. Trust-weighted rating | Partial: Return Rate unavailable; labeled "Adjusted Trust Score" |
| 6. Dynamic price banding | Complete; dynamic behavior verified |

## Data Limitations
The dataset contains no Sales, Stock Value, Return Rate or Promotion Flag fields, and the scraper never collected them. Nothing was estimated or substituted. See `Task_Verification_Report.md` for details.
