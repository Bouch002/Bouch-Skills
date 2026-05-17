# Investment Benchmarks

Reference thresholds for interpreting `property-investment-report` outputs.

---

## Gross yield benchmarks

These are typical gross yield ranges for UK residential BTL. Use them to contextualise the calculated yield in the report verdict.

| Yield | Verdict |
|---|---|
| 7%+ | Strong — above average for most UK markets |
| 5–7% | Good — acceptable for most investors |
| 4–5% | Marginal — viable in low-growth markets, thin in high-value areas |
| Below 4% | Weak — difficult to stack unless significant capital growth expected |

**Regional context:** Higher-value markets (London, South East) typically yield 3–5% gross. Northern cities (Manchester, Leeds, Liverpool, Sheffield) typically yield 5–8%. HMOs typically yield 8–12% gross but carry higher management and compliance costs.

---

## Net yield deduction methodology

The skill uses a flat ~25% cost assumption for the net yield estimate. Break this down for the user if they ask:

| Cost | Typical range |
|---|---|
| Letting agent management | 10–12% of rent |
| Maintenance and repairs | 8–10% of rent |
| Void periods | 3–5% of rent |
| **Total** | **~21–27%** |

**Adjustments:**
- Self-managed: remove management fee → use ~13–15% deduction instead
- HMO / licensed property: add 3–5% for licensing compliance and higher turnover
- New build: reduce maintenance to ~4–6% for first 5 years (NHBC warranty)

---

## EPC rating scale and rental compliance

| Rating | Score | Rental status |
|---|---|---|
| A | 92–100 | Compliant — excellent efficiency |
| B | 81–91 | Compliant — very good |
| C | 69–80 | Compliant — meets proposed 2028 minimum |
| D | 55–68 | Compliant today; upgrade needed by 2028 |
| E | 39–54 | Currently legal to let; at risk post-2028 |
| F | 21–38 | Illegal to let new tenancies (since 2020) |
| G | 1–20 | Illegal to let (since 2020) |

**Key regulation dates:**
- From April 2020: minimum EPC E for new tenancies
- From April 2023: minimum EPC E for all existing tenancies
- **Proposed from 2028:** minimum EPC C for new tenancies (not yet law as of 2025 — monitor for updates)

Typical upgrade costs:
- D → C: ~£3,000–£8,000 (insulation, boiler upgrade, smart controls)
- E → C: ~£8,000–£15,000
- F/G → C: £15,000–£25,000+ (may require full retrofit)

---

## Value vs. comparables

| Asking vs. comparable median | Signal |
|---|---|
| >10% below | Potential undervalue — verify condition and reason |
| Within ±5% | Fair market pricing |
| 5–10% above | Premium — check for justification (condition, size, EPC) |
| >10% above | Overpriced relative to local market — yield will be compressed |

---

## SDLT quick reference (additional property / BTL)

The `stamp_duty` tool computes the exact figure. These thresholds are for quick orientation only. Additional property surcharge is currently **5%** on top of standard rates (from October 2024).

| Purchase price | Approx. SDLT (BTL / second home) |
|---|---|
| £150,000 | ~£7,500 |
| £200,000 | ~£10,000 |
| £250,000 | ~£13,750 |
| £300,000 | ~£17,000 |
| £400,000 | ~£27,000 |
| £500,000 | ~£37,500 |

Always use the `stamp_duty` tool result — these figures are indicative only.
