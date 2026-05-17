# UK Property Skills

Tracks built and planned skills using `uk-property-mcp` / `property-shared`.

---

## Built

### `property-investment-report` — 2026-05-17 (v2 same day)

Full buy-to-let investment analysis with listing-first workflow and conditional deep-dive. Phase 1 reads the Rightmove listing before running any financial analysis. Phase 2 runs comparables, yield, rental, and SDLT. Automatically escalates to PPD history + market ceiling search when positive investment signals are detected.
Tools: `rightmove_search` → `rightmove_listing` → `property_report` + `rental_analysis` + `stamp_duty` → (conditional) `ppd_transactions` + `rightmove_search` (price_high)

---

## Planned / Ideas

### `property-block-analysis`

Screen a postcode for block-buy opportunities.
Tools: `property_blocks` → `property_comps` → `property_yield`
Output: ranked block opportunities with yield and price/sqft benchmarks.

### `planning-research`

Planning history and site research for a given address or postcode.
Tools: `planning_search` → `ppd_transactions` → `property_epc`
Output: planning applications timeline, transaction history, EPC rating trend.

### `company-profile`

Deep-dive on a developer, landlord, or property company.
Tools: `company_search` → `company_profile` → `ppd_transactions`
Output: company overview, portfolio transaction history.

### `listing-analysis`

Analyse a specific Rightmove listing against local comparables.
Tools: `rightmove_listing` → `property_comps` → `property_epc` → `rental_analysis`
Output: is it priced fairly, yield if rented, EPC context.
