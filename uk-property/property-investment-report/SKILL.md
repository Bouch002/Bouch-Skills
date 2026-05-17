---
name: property-investment-report
description: Full buy-to-let investment analysis for a UK residential property. Use when evaluating a property for rental investment, calculating yield, estimating stamp duty, assessing comparable sales, or producing a purchase decision report. Covers gross yield, net yield estimate, comparable sales, rental market context, EPC rating, and SDLT cost in one structured output. Automatically escalates to deep-dive analysis (PPD history, market ceiling) when positive investment signals are detected.
compatibility: Requires uk-property-mcp MCP server
metadata:
  author: bouch
  version: "2.2"
allowed-tools: mcp__claude_ai_uk-property-mcp__rightmove_search mcp__claude_ai_uk-property-mcp__rightmove_listing mcp__claude_ai_uk-property-mcp__property_report mcp__claude_ai_uk-property-mcp__rental_analysis mcp__claude_ai_uk-property-mcp__stamp_duty mcp__claude_ai_uk-property-mcp__ppd_transactions
---

## Overview

A two-phase workflow. Phase 1 identifies and understands the asset before any financial analysis runs. Phase 2 pulls the financial data. An automatic deep-dive triggers if the property meets positive investment criteria — surfacing PPD transaction history and market ceiling without waiting to be asked.

## Required inputs

The skill accepts a Rightmove URL **or** a street address + postcode. Provide one of:

| Option | Example |
|---|---|
| Rightmove URL | `https://www.rightmove.co.uk/properties/12345678` |
| Street address + postcode | `10 High Street, Derby, DE1 2AB` |

Additional required inputs:

| Input | Notes |
|---|---|
| Purchase price | Asking price or agreed price in £. If a guide range is given, use the midpoint. |

**Assumed defaults (no need to ask):**
- Buyer is an investor — SDLT calculated at additional property rates
- Buyer has no onward chain — report assumes unencumbered purchase
- Both **cash** and **BTL mortgage** scenarios are always shown

**Optional:** bedrooms, property type — used to refine rental estimates.

If the Rightmove URL or address is the only input provided, extract price, postcode, and property details from the listing in Phase 1 before proceeding.

---

## Phase 1 — Understand the asset

### Step 1 — Find the listing

**If a Rightmove URL or listing ID is provided:**
Call `rightmove_listing` directly with the URL or ID. Skip `rightmove_search` entirely.

**If a street address + postcode is provided:**
Call `rightmove_search` with the postcode at 0.25 mile radius. Match the property by address and price, then call `rightmove_listing` with the matched listing ID for full detail.

If no Rightmove listing is found (off-market, recently agreed, or no match), note it and proceed to Phase 2 using the address/postcode provided. Flag that listing-level data is unavailable.

### Step 2 — Read the full listing

Read the full listing description carefully.

**Before proceeding, establish:**

- **Property type** — is this a finished residential property, an unconverted structure (barn, shell, plot), a commercial building, or a listed / restricted asset? If non-standard, flag clearly and adjust the entire analysis accordingly.
- **Condition signals** — language like "heaps of potential", "scope for improvement", "sold as seen", "in need of modernisation", or "probate sale" signals works needed. Note these explicitly.
- **Size and layout** — bedrooms, bathrooms, floor area if stated. Flag if floor area seems large relative to the asking price (potential undervalue) or small (potential overvalue).
- **Time on market** — `first_visible_date` tells you how long it has been listed. Over 3 months warrants a note; over 6 months is a material signal.
- **Listing agent** — useful context for negotiation and local market knowledge.

**Do not proceed to Phase 2 until you have a clear picture of what is being purchased.**

### Data sufficiency check

After reading the listing, confirm what data is available and what degrades without it.

| Data point | Source | Needed for | Available from URL/listing? |
|---|---|---|---|
| Postcode | `rightmove_listing` returns it | `rental_analysis`, `ppd_transactions`, `property_report` | ✓ Always |
| Street name | `rightmove_listing` returns it | `property_report`, `ppd_transactions` | ✓ Always |
| Purchase price | `rightmove_listing` returns it | `stamp_duty`, yield calculations | ✓ Always |
| House number / name | **Not returned by Rightmove** | Accurate EPC match, specific PPD lookup | ✗ Usually missing |

**If the house number is missing**, two things degrade:

1. **EPC data** — `property_report` will likely match the EPC of a different property at the same postcode. Flag this in the report and treat the EPC rating as unconfirmed unless it matches the listing's stated EPC.
2. **PPD (specific property)** — without a house number, `ppd_transactions` returns the full street history rather than this property's history. Still useful, but note the limitation.

**Ask for the house number using this prompt if not already provided:**

> *"To confirm the EPC rating and look up this property's specific transaction history, could you provide the house number or name? (e.g. '42' or 'Rose Cottage'). It's not always shown on Rightmove. If you don't have it, I can proceed — I'll note where data is estimated or based on a nearby property."*

Proceed without the house number if the user cannot provide it or does not respond. Do not block the report.

---

## Phase 2 — Financial analysis

Run all three tool calls in parallel:

### Step 3 — Pull full property data

Call `property_report` with the street name and postcode (house number optional but improves EPC matching).

**Always cross-check the returned EPC against the listing:**
- Compare the EPC floor area and property type from `property_report` against the listing's stated size and type
- If they match — use the EPC data as confirmed
- If they differ significantly (e.g. report shows a 70sqm bungalow but listing shows a 132sqm semi) — the EPC belongs to a different property at the postcode. Follow this resolution order:

  1. **Ask the user:**
     > *"The EPC data we retrieved appears to belong to a different property at this postcode. Do you know the EPC rating for this property? (A–G, e.g. 'C'). You may be able to find it on the listing or from the agent."*
  2. **If the user provides a rating** — use it and note it was supplied by the user
  3. **If the user says they don't know** — fall back to the EPC rating stated in the Rightmove listing key features (most listings include it)
  4. **If the listing doesn't state an EPC either** — note as unconfirmed and flag as a due diligence item before purchase

- This mismatch is common when only a street name is available; it is not an error, just a data limitation

### Step 4 — Get rental market figures

Call `rental_analysis` with postcode and purchase price. Note if the comparables appear to be a different size or type from the subject property — the yield figure will be misleading if so.

### Step 5 — Calculate SDLT

Call `stamp_duty` with purchase price and buyer status. Set `additional_property: true` for BTL / second-home purchases.

---

## Phase 3 — Score and escalate

### Step 6 — Score against investment criteria

After Steps 3–5, check the property against the escalation triggers below. If **any** trigger is met, automatically proceed to the deep-dive in Step 7 without waiting to be asked.

**Escalation triggers:**

| Trigger | Threshold |
|---|---|
| Price vs. area sold median | Asking price >10% below |
| Time on market | Listed >6 months |
| No PPD history visible | Property appears never to have transacted in digital records |
| Floor area vs. price | Large property (>150sqm) priced at or below area median |
| Gross yield | >5.5% — genuine yield signal worth verifying |
| EPC gap | Rating D or below but potential B or C — upgrade plays |
| Listing language | "Potential", "probate", "sold as seen", "development opportunity" |

### Step 7 — Deep-dive (auto-triggered)

Run both calls in parallel:

**PPD transaction history** — call `ppd_transactions` with the street, postcode, and house number/name. This reveals:
- What the property last sold for and when — establishes true held value
- What other properties on the same street have transacted for — establishes the real price ceiling
- Long hold periods (no digital record) — often signals underpricing or estate sale

**Market ceiling** — call `rightmove_search` with `sort_by: price_high` and a 1–1.5 mile radius. This shows the top of the local market and what a significantly improved or larger version of the property could achieve.

---

## Data confidence ratings

Every section in the report is assigned a traffic light rating reflecting how well the underlying data has been verified. Apply these consistently.

| Rating | Meaning | When to use |
|---|---|---|
| 🟢 Confirmed | Data sourced directly from MCP tools and verified to match this property | EPC matches listing size/type; PPD found for this specific address; rental comps match property type |
| 🟡 Estimated | Data is approximate, area-level, or sourced from a proxy | EPC from listing key features (not MCP-confirmed); rental comps are a different size/type; PPD is street-level only |
| 🔴 Unconfirmed | Data is missing, could not be retrieved, or is known to be from a different property | EPC mismatch with no listing fallback; no rental listings found; house number not provided |

**Re-run note** — include the following for any 🟡 or 🔴 section, adapted to the specific gap:

> *📋 Re-run with [house number / confirmed EPC / house number for specific PPD] to confirm this figure.*

Assign the rating at the section level in the report header, not line by line.

---

## Calculations

**Gross yield** = (annual rent ÷ purchase price) × 100

Use the median from `rental_analysis`. If the comparables are clearly a different property type or size, note that the figure is unreliable and provide a bracketed range using estimated market rent instead.

**Net yield estimate** = gross yield × 0.75

Assumes ~25% costs. Adjust narrative for self-managed (×0.87) or HMO (×0.68). See [references/benchmarks.md](references/benchmarks.md).

**Total acquisition cost** = purchase price + SDLT + estimated legal and survey (~£2,500–£4,000).

---

## Output format

```
## Investment Report: [Address], [Postcode]
*[Date]*

**Data confidence:** 🟢 Confirmed  🟡 Estimated  🔴 Unconfirmed

---

### What is being purchased
[1–2 sentences from Phase 1: property type, condition, time on market.
Flag clearly if non-residential, unconverted, or restricted.]

### Property Overview [🟢 / 🟡 / 🔴]
- **Type / size:** [from listing + EPC]
- **EPC rating:** [A–G] — [compliance status vs 2028 C requirement]
  - 🟢 if EPC confirmed from MCP tools (floor area + type match listing)
  - 🟡 if taken from listing key features — *📋 Re-run with house number for MCP-confirmed EPC*
  - 🔴 if unknown — *📋 Request EPC from agent or check the Energy Performance Certificate register*
- **Purchase price:** £[X] (guide: £[low]–£[high] if range given)
- **Listed since:** [date] ([X days/months] on market)
- **Tenure:** [Freehold / Leasehold]
- **Council tax band:** [X]

### Market Value Context [🟢 / 🟡]
- Area sold median (Land Registry, 24 months): £[X]
- Current asking median (Rightmove, local): £[X]
- Asking price is [X]% [above / below] sold median
- [1-line market commentary]
- 🟡 if fewer than 10 transactions in the period — *📋 Re-run with a wider date range for more robust median*

### Rental Analysis [🟢 / 🟡 / 🔴]
- Comparables: [N] listings within [X] miles, median £[X]/month
- Annual rental income (median): £[X]
- **Gross yield: [X.X]%** — [Strong / Good / Marginal / Weak]
- **Net yield estimate: [X.X]%** (~25% cost assumption)
- 🟢 if comparables match property type and size
- 🟡 if comparables are a different size/type, or radius had to be widened — *📋 Re-run with bedrooms and property type specified, or obtain a lettings agent valuation*
- 🔴 if no comparables found — *📋 Rental market data unavailable; obtain a lettings agent valuation before proceeding*

### Scenario A — Cash Purchase [🟢]
- Total cash deployed: £[purchase] + £[SDLT] + £[legal/survey] = **£[total]**
- Annual net income estimate: **£[X]** *(inherits rental confidence rating above)*
- Net yield on capital: **[X.X]%**

### Scenario B — BTL Mortgage (25% deposit) [🟢 / 🟡]
- Deposit: £[X] | Mortgage: £[X] (75% LTV)
- Monthly interest (~5.0% indicative): £[X]
- ICR at median rent: [X]% — [Pass / Fail at 125% threshold]
- Monthly cash flow estimate: £[X] surplus / (£[X] shortfall)
- 🟡 if lender restrictions apply — *📋 Speak to a specialist broker before proceeding*
- [Flag any restrictions: commercial use, non-standard construction, short lease, etc.]

### Purchase Costs [🟢]
- SDLT (additional property): £[X] ([X]% effective rate)
- Estimated legal / survey: ~£[X]
- **Total acquisition cost (cash): ~£[X]**

### Deep-Dive Findings [🟢 / 🟡]
[Only present if Step 7 was triggered.]
- PPD: 🟢 if house number confirmed and specific property found | 🟡 if street-level only — *📋 Re-run with house number for this property's specific transaction history*
- [PPD history table + market ceiling summary]

### Investment Verdict
[3–4 sentences. Lead with the asset reality and condition. Benchmark yield.
Note any sections rated 🟡 or 🔴 that materially affect the verdict.
State clearly whether this is a yield play, value/capital play, development
opportunity, or not investable in current form.]

---
*Report assumes an investment purchase with no onward chain. Timelines may be
longer than estimated if a chain is subsequently identified, or if information
provided differs from the legal title or survey findings. Sections rated 🟡 or
🔴 are based on estimated or unverified data — re-run with the information
indicated to confirm. This report is for informational purposes only and does
not constitute financial or legal advice.*
```

---

## Risk flags

Always surface in the Verdict:

- **Non-residential / unconverted asset** — rental and yield data will be meaningless
- **EPC E, F, or G** — rental compliance risk from 2028; estimate upgrade cost
- **Yield below 4%** — weak; see [references/benchmarks.md](references/benchmarks.md)
- **Asking price >10% above comparable median** — yield compression
- **Thin or mismatched rental market** — flag if comparables are the wrong property type
- **Stale listing (>6 months)** — why hasn't it sold? Investigate before proceeding
- **No PPD history** — potential long hold; negotiate accordingly
