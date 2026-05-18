---
name: natal-reading
description: Full natal chart interpretation for a birth date, time, and place. Use when someone shares their birth data and wants an astrological reading, natal chart analysis, birth chart interpretation, or wants to understand their Sun sign, Moon sign, Rising sign, planets, houses, or aspects. Covers the Big Three, personal planets, chart ruler, major aspects, and house themes in one structured output. Keywords: natal chart, birth chart, astrology reading, horoscope, Sun Moon Rising, astrological placements.
compatibility: Requires Celestine MCP server (Swiss Ephemeris)
metadata:
  author: bouch
  version: "1.0"
allowed-tools: mcp__claude_ai_Celestine-mcp__birth_chart mcp__claude_ai_Celestine-mcp__get_sign mcp__claude_ai_Celestine-mcp__get_planet mcp__claude_ai_Celestine-mcp__get_house mcp__claude_ai_Celestine-mcp__get_aspect
---

## Overview

A three-phase workflow: calculate the natal chart, enrich key placements with sign/planet/house/aspect archetypes, then synthesise into a structured reading. Covers the Big Three (Sun, Moon, Ascendant), personal planets, chart ruler, major aspects, and dominant house themes.

Never compute planetary positions or aspects manually — always call `birth_chart` for the data.

## Required inputs

| Input | Notes |
|---|---|
| Date of birth | Day, month, year |
| Place of birth | City and country (or coordinates) |
| Time of birth | Ask explicitly — affects Ascendant, houses, and Moon precision |

**If birth time is unknown:** proceed without it. Omit the Ascendant, Chart Ruler, and House Themes sections. Note that Moon sign may be uncertain if born near a sign boundary. Do not block the reading — deliver everything else in full.

---

## Phase 1 — Gather inputs and calculate the chart

### Step 1 — Collect birth data

If date and place are provided but time is missing, ask once:

> *"Do you know your birth time? Even an approximate time helps calculate your Ascendant and house placements — these add significant depth to the reading. If you don't have it, no problem, I can still produce a strong reading from your date and place alone."*

Do not ask a second time if the user says they don't know.

### Step 2 — Calculate the natal chart

Call `birth_chart` with `detail=full`.

- With birth time → full chart including Ascendant, Midheaven, and house cusps
- Without birth time → full chart without houses; note the limitation clearly in the output header

---

## Phase 2 — Resource enrichment

After receiving the chart data, run all resource reads in parallel.

### Step 3 — Read sign archetypes

Call `get_sign` for:
- Sun sign
- Moon sign
- Ascendant sign (only if birth time known)

### Step 4 — Read planet archetypes

Call `get_planet` for:
- Chart ruler (the planet ruling the Ascendant sign — see [references/interpretation-guide.md](references/interpretation-guide.md) for the ruler lookup table)
- Any planet involved in a stellium (3+ planets in one sign or house), if present

### Step 5 — Read house meanings

Call `get_house` for:
- Houses 1 and 10 (always — identity and public role)
- Any house containing a stellium (3+ planets)
- The house the chart ruler occupies

Only call `get_house` if birth time is known.

### Step 6 — Read aspect archetypes

From the chart, identify the top 3–5 aspects by significance. Prioritise:

1. Conjunctions involving Sun, Moon, Ascendant, or chart ruler
2. Oppositions and squares between personal planets
3. Trines and sextiles involving a luminary

Call `get_aspect` for each selected aspect type (e.g. `conjunction`, `opposition`, `square`, `trine`, `sextile`).

See [references/interpretation-guide.md](references/interpretation-guide.md) for orb thresholds and selection guidance.

---

## Phase 3 — Synthesise and output

Combine chart data and resource archetypes into the output format below. Write in the second person ("your Sun in Scorpio...") for a personal reading, or third person if the subject is someone else.

---

## Output format

```
## Natal Chart: [Name / "Your Chart"]
*Born: [date] at [time] in [place]*
[If birth time unknown: "*Birth time not provided — Ascendant, chart ruler, and house themes are unavailable.*"]

---

### The Big Three

**Sun in [Sign][, House N if known]**
[2–3 sentences on core identity, life purpose, and where vitality naturally flows — draw from Sun archetype and sign qualities]

**Moon in [Sign][, House N if known]**
[2–3 sentences on emotional nature, instincts, and what the person needs to feel secure]

**[Sign] Rising**
[2–3 sentences on outer personality, first impressions, and how the world initially meets them]
*Chart Ruler: [Planet] in [Sign], House [N] — [one sentence on what this emphasises in the chart]*

*[Omit the Rising and Chart Ruler block entirely if birth time is unknown]*

---

### Chart Overview

| | |
|---|---|
| Element balance | Fire [n] · Earth [n] · Air [n] · Water [n] |
| Modality balance | Cardinal [n] · Fixed [n] · Mutable [n] |

[1–2 sentences interpreting the dominant element and modality — what quality the chart leads with]
[Include hemisphere emphasis only if 7+ planets fall in one hemisphere and birth time is known]

---

### Personal Planets

**Mercury in [Sign][, House N]** — [1–2 sentences on communication style, how they think and process information]

**Venus in [Sign][, House N]** — [1–2 sentences on love language, values, aesthetic sense, and what they attract]

**Mars in [Sign][, House N]** — [1–2 sentences on drive, action style, and how they assert themselves]

---

### Outer Planets

**Jupiter in [Sign][, House N]** — [1 sentence — the area of life where growth and expansion is most available]

**Saturn in [Sign][, House N]** — [1 sentence — the area of life requiring discipline, patience, and earned mastery]

*Uranus, Neptune, and Pluto operate at a generational level; interpret by house placement where birth time is known, otherwise note sign only.*

---

### Major Aspects

[For each of the top 3–5 aspects:]

**[Planet] [aspect] [Planet]** (orb: [X]°)
[1–2 sentences interpreting this aspect — use the aspect archetype combined with the nature of the two planets]

---

### House Themes
*[Omit entirely if birth time is unknown]*

[For any house with 2+ planets — name the house theme and note the emphasis]
[Always address the 1st house (self and appearance), 7th (relationships and partnerships), and 10th (career and public role) if occupied]
[Name any stellium house and interpret the concentrated energy there]

---

### Reading Summary

[3–4 sentences synthesising the chart's dominant signature: the loudest element/modality, the most active or emphasised planet, and the through-line connecting the Big Three to the major aspects. Identify 1–2 overarching themes this chart tends to return to.]

---
*Planetary positions calculated via Swiss Ephemeris through the Celestine MCP. House placements and the Ascendant require a confirmed birth time — sections are omitted where birth time is unknown. This reading is for reflective purposes only.*
```

---

## Edge cases

**Stellium (3+ planets in one sign or house):** Name and interpret it as a concentrated theme. Address each planet within it briefly rather than in isolation — they operate as a cluster.

**Unknown birth time:** Omit Ascendant, Chart Ruler, and House Themes entirely. Do not estimate or guess houses. Deliver Sun, Moon, personal planets, outer planets, and aspects in full.

**Retrograde planets:** Note retrograde (Rx) status inline in the relevant planet's paragraph — do not list it separately. Example: "Mercury Rx here suggests a more inward, revisionary style of thinking."

**Unaspected planet:** A planet with no major aspects operates independently. Note it as a "singleton energy" — autonomous, sometimes unpredictable, but capable of great focus in its domain.

**Moon near a sign boundary:** If the birth chart returns the Moon near the end or start of a sign (within ~1°) and no birth time is given, note that the Moon sign is uncertain and provide a brief reading for both candidate signs.
