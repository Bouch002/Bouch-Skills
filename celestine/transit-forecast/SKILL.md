---
name: transit-forecast
description: Planetary transit forecast for a birth chart over a user-specified date range. Use when someone asks about upcoming transits, their astrological forecast, what's happening in their chart, planetary influences over a period, key astrological dates, or what to expect astrologically. Covers all significant transiting planets aspecting natal positions, ranked by impact, with key dates and period themes. Keywords: transits, transit forecast, astrological forecast, upcoming transits, planetary influences, what's in my chart, astrology this month, what's happening astrologically, what planets are affecting me.
compatibility: Requires Celestine MCP server (Swiss Ephemeris)
metadata:
  author: bouch
  version: "1.0"
allowed-tools: mcp__claude_ai_Celestine-mcp__transit_search mcp__claude_ai_Celestine-mcp__current_transits mcp__claude_ai_Celestine-mcp__get_aspect mcp__claude_ai_Celestine-mcp__get_planet mcp__claude_ai_Celestine-mcp__get_sign
---

## Overview

A three-phase workflow: calculate transits over a date range using `transit_search`, enrich the most significant transits with planet and aspect archetypes, then synthesise into a structured forecast with ranked transits, period themes, and key dates to watch.

Never compute transit data manually — always call `transit_search` or `current_transits`.

## Required inputs

| Input | Notes |
|---|---|
| Date of birth | Day, month, year |
| Place of birth | City and country (or coordinates) |
| Time of birth | Ask — affects transits to Ascendant, Midheaven, and house cusps |
| Forecast period | Date range or relative window ("next month", "next 3 months") |

**Default period:** If no date range is given, forecast the next 30 days from today's date.

**If birth time is unknown:** proceed without it. Transits to Ascendant, Midheaven, and house cusps cannot be calculated. Note this once in the output header. Do not block the forecast.

---

## Phase 1 — Gather inputs

### Step 1 — Collect birth data and period

If birth time is missing, ask once:

> *"Do you know your birth time? It allows me to include transits to your Ascendant, Midheaven, and house cusps — these add meaningful precision to the forecast. If not, I can still calculate all transits to your natal planets."*

Do not ask a second time. Convert relative date ranges to absolute dates before calling the MCP (e.g. "next month" → first and last day of the next calendar month).

---

## Phase 2 — Calculate transits

### Step 2 — Run transit search

Call `transit_search` with:
- Birth date, place, and time (if known)
- Start and end dates of the forecast period
- `mode=full`

### Step 3 — Current transits snapshot (conditional)

If today falls within the forecast period, also call `current_transits` with `mode=summary`. Use this to confirm which transits are active right now and note current orb — active transits get higher prominence in the output.

---

## Phase 3 — Resource enrichment

Run all resource reads in parallel after receiving transit data.

### Step 4 — Identify significant transits

Apply the significance ranking from [references/transit-interpretation-guide.md](references/transit-interpretation-guide.md):

1. Outer-planet transits (Pluto, Neptune, Uranus, Saturn, Jupiter) to personal natal planets (Sun, Moon, Mercury, Venus, Mars) and angles (Ascendant, Midheaven)
2. Conjunctions and oppositions rank above squares; squares above trines and sextiles
3. Tighter orbs rank higher within each tier
4. Multi-hit transits (3 exact hits due to retrograde) are flagged with all three dates

Cap the Major Transits section at 8 entries. If there are more, list remaining ones in the condensed Additional Active Transits table.

### Step 5 — Read aspect archetypes

Call `get_aspect` for each distinct aspect type appearing in the top 8 transits (e.g. `conjunction`, `opposition`, `square`, `trine`). Do not repeat calls for the same aspect type.

### Step 6 — Read planet archetypes

Call `get_planet` for each distinct transiting planet in the top 8 transits. Do not repeat calls for the same planet.

### Step 7 — Read sign archetypes (conditional)

If a major planet (Saturn, Jupiter, Uranus, Neptune, or Pluto) ingresses into a new sign during the forecast period, call `get_sign` for the new sign. Ingresses represent a collective shift and belong in the Period Overview.

---

## Phase 4 — Synthesise and output

Combine transit data and resource archetypes into the output format below. Interpret transits in the second person ("Saturn is squaring your natal Venus...") unless the subject is someone else.

For each major transit, address:
- What the transiting planet is activating in the natal chart (the natal planet's core theme)
- The nature of the aspect (conjunction = new cycle; opposition = tension/awareness; square = friction/growth; trine = flow/ease)
- The real-world area of life this is likely to touch
- How long this energy is active and when it peaks (exact date)

---

## Output format

```
## Transit Forecast: [Name / "Your Transits"]
*Period: [start date] – [end date] | Born: [birth data]*
[If birth time unknown: "*Birth time not provided — transits to Ascendant, Midheaven, and house cusps are excluded.*"]

---

### Period Overview

[2–3 sentences naming the dominant planetary energy of this period and the overarching theme it creates. Reference the most significant transit by name. If a major ingress is happening, flag it here.]

---

### Major Transits

**[Transiting Planet] [aspect] natal [Natal Planet]**
Exact: [date] [· [date] · [date] if multi-hit] · Active: [duration window]
[2–3 sentences: what this transit activates, how the aspect shapes it, and the likely real-world territory it touches. Ground it in practical terms.]

[Repeat for each of the top 8 transits, in significance order]

---

### Additional Active Transits

| Transit | Exact | Nature |
|---|---|---|
| [Planet] [aspect] natal [Planet] | [date] | [one-line theme] |

[Omit this section entirely if there are 8 or fewer total significant transits]

---

### Key Dates

[Chronological list of exact transit dates, ingress dates, and station points within the period.]
**[Date]** — [what's happening in one line]

---

### Themes for the Period

[3–4 sentences pulling together the dominant threads across all active transits. Name the 1–2 overarching life areas under pressure or flow. Note if the period contains a concentration of one type of energy (e.g. all squares = a pressure-testing period; multiple conjunctions = new beginnings).]

---
*Planetary positions calculated via Swiss Ephemeris through the Celestine MCP. Transits to angles and house cusps require a confirmed birth time. This forecast is for reflective purposes only.*
```

---

## Edge cases

**No birth time:** Omit all transits to Ascendant, Midheaven, and house cusps. Note once at the top. Do not omit any planetary transits.

**Very short forecast window (< 7 days):** Supplement `transit_search` with `current_transits` to get orb precision for fast-moving planets. Include Mercury, Venus, and Sun transits that are active within the window even if they would normally be lower priority.

**Very long forecast window (> 6 months):** Focus Major Transits on outer-planet transits (Saturn, Uranus, Neptune, Pluto) and Jupiter. Fast-moving planet transits are too numerous — exclude Sun, Mercury, Venus, Mars from Major Transits and note they are not included.

**Multi-hit transit (retrograde):** Note all three exact dates. Label the retrograde period between hits 1 and 3 as a time of review and internal processing. Flag that this transit's theme is likely to resurface and deepen across the full retrograde cycle.

**Transiting planet stationing near natal planet:** When a transiting planet stations (turns retrograde or direct) within 2° of a natal planet, the transit's intensity is dramatically extended. Flag it explicitly: *"Saturn stations retrograde at [X]° — very close to your natal [Planet] at [Y]°. This period is particularly intense."*

**No significant outer-planet transits:** If a period returns no Tier 1 or Tier 2 transits, deliver the available fast-planet transits and note: *"This is a relatively quiet outer-planet period — the transits present are shorter-cycle influences."*
