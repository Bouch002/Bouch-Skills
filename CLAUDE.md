# Bouch Skills — Claude Instructions

This project builds Agent Skills conforming to the [agentskills.io](https://agentskills.io) specification. Each skill wraps one or more Bouch MCP servers into a reusable, structured workflow.

## Project structure

```
Bouch-Skills/
├── CLAUDE.md                          # This file
├── AgentSkills.io doc structure...md  # Full agentskills.io spec reference
├── uk-property/
│   ├── BUILDS.md                      # Build log and idea backlog
│   └── <skill-name>/                  # One folder per built skill
│       ├── SKILL.md
│       ├── references/
│       └── assets/
├── celestine/
│   ├── BUILDS.md
│   └── <skill-name>/
└── uk-archives/
    ├── BUILDS.md
    └── <skill-name>/
```

## Skill domains and their MCPs

### uk-property
Uses `uk-property-mcp` (preferred) and `property-shared`.

| Tool | Purpose |
|---|---|
| `property_report` | Full data pull for an address + postcode |
| `property_comps` | Comparable sales with EPC-enriched price/sqft |
| `property_yield` | Yield data for a postcode |
| `ppd_transactions` | Price paid history |
| `rightmove_search` | Browse current listings |
| `rightmove_listing` | Full detail on one listing |
| `property_epc` | Energy certificates |
| `rental_analysis` | Rental market figures |
| `stamp_duty` | SDLT calculation |
| `property_blocks` | Block-buy opportunities |
| `planning_search` | Council planning portals |
| `company_search` + `company_profile` | Developer / landlord lookup |

### celestine
Uses the Celestine MCP (Swiss Ephemeris). **Never compute astrological data manually** — always call the MCP tools. When a tool returns a `download_url`, surface it verbatim.

| Tool | Purpose |
|---|---|
| `birth_chart` | Full natal chart (detail=full or compact) |
| `current_transits` | Active transits at a moment |
| `transit_search` | Transits over a date range |
| `planetary_positions` | Sky positions for any date |
| `progression_report` | Secondary progressions + solar arc |
| `generate_calendar_events` | Astrological calendar with .ics download |
| `get_planet` | Planet archetype + rulership lookup |
| `get_sign` | Sign qualities, themes, shadow lookup |
| `get_house` | House meanings lookup (1–12) |
| `get_aspect` | Aspect angle, orb, polarity lookup |

### uk-archives
MCP / data source TBC. Update this section once confirmed.

## BUILDS.md convention

Each domain folder contains a `BUILDS.md` with two sections:

- **Built** — completed skills, listed with name, a one-line description, and the date shipped
- **Planned / Ideas** — candidate skills with name, purpose, and the expected MCP tool chain

When a skill is built, move its entry from Planned → Built and add the ship date. Add new ideas freely to the Planned section.

## Building a skill

Follow the agentskills.io spec (full reference: `AgentSkills.io doc structure for skills.md`).

### Rules
- Skill folder name = `name` field in frontmatter (lowercase, hyphens only, no consecutive hyphens)
- `name` and `description` are required; description must be keyword-rich (agents use it to activate the skill)
- Keep `SKILL.md` under 500 lines — move detail to `references/` files
- List required MCP servers in the `compatibility` field
- Use `allowed-tools` to pre-approve MCP tool calls the skill will always make

### Minimal SKILL.md template

```markdown
---
name: skill-name
description: What it does and when to use it. Include keywords for agent matching.
compatibility: Requires uk-property-mcp MCP server
metadata:
  author: bouch
  version: "1.0"
allowed-tools: mcp__claude_ai_uk-property-mcp__property_report mcp__claude_ai_uk-property-mcp__property_yield
---

## Overview

One paragraph describing the skill's purpose and output.

## Steps

1. ...
2. ...

## Output format

Describe the expected output structure here.
```

### Progressive disclosure pattern
1. `name` + `description` load at agent startup — keep them sharp
2. Full `SKILL.md` body loads on activation — keep it concise
3. `references/` and `assets/` files load on demand — push detail here

## Skill workflow patterns

These patterns have been established through building `property-investment-report` and should be reused in future skills where applicable.

### Listing-first pattern (uk-property)

Always understand the asset before running financial analysis. Call `rightmove_listing` (via URL or ID from `rightmove_search`) as the first step. Establish property type, condition, size, and time on market before calling `property_report`, `rental_analysis`, or `stamp_duty`. A barn with planning permission and a finished 3-bed semi return identical postcode-level financial data — the listing is what differentiates them.

### Data sufficiency check

After Phase 1, check what data is available before calling Phase 2 tools. Document what degrades without a house number:

- `rental_analysis` and `stamp_duty` — fully functional from postcode + price alone
- `property_report` — works but EPC may match a different property at the postcode
- `ppd_transactions` — works at street level without house number; specific property history needs it

If the house number is missing, prompt the user before Phase 2. Offer to proceed without it if they can't provide it — never block the report.

### Conditional deep-dive / escalation

Define escalation triggers in the skill. If any trigger fires after Phase 2, auto-call `ppd_transactions` and `rightmove_search` (price_high sort) without waiting to be asked. Triggers for uk-property skills: price >10% below area median, listed >6 months, no PPD history, large floor area at or below median price, gross yield >5.5%, EPC D or below with upgrade potential, or "potential/probate/development" listing language.

### Dual scenario output (investment skills)

Always present both a **cash purchase** and a **BTL mortgage** scenario. Calculate the mortgage ICR and flag if the property fails the 125% threshold. Flag commercial elements, non-standard construction, or short leases that restrict BTL mortgage products.

### Investor defaults

For investment-focused skills, apply these without asking:

- SDLT at additional property rates
- No onward chain assumed
- Purchase price = midpoint of guide range if a range is given

## Report conventions

### Traffic light rating system

Every section in a report gets a confidence rating:

- 🟢 **Confirmed** — data sourced from MCP tools and verified to match the subject property
- 🟡 **Estimated** — data is approximate, area-level, from a proxy, or the comparables don't match the property type/size
- 🔴 **Unconfirmed** — data is missing or known to be from a different property

Include a `📋 Re-run with [specific missing data] to confirm this figure.` note on every 🟡 or 🔴 section.

### Standard report footer

End every investment report with:

> *Report assumes an investment purchase with no onward chain. Timelines may be longer than estimated if a chain is subsequently identified, or if information provided differs from the legal title or survey findings. Sections rated 🟡 or 🔴 are based on estimated or unverified data — re-run with the information indicated to confirm. This report is for informational purposes only and does not constitute financial or legal advice.*

### EPC mismatch handling

`property_report` matches EPC by postcode sector, not by house number. For a postcode with mixed property types, the returned EPC often belongs to a different property. Always compare the EPC floor area and property type against the listing. If they differ: (1) ask the user if they know the EPC, (2) fall back to the EPC stated in the listing key features, (3) if neither is available, mark as 🔴 Unconfirmed.

## Conventions

- Place all interpretation guidance (e.g. EPC rating scales, yield benchmarks, astrological orbs) in `references/` not in `SKILL.md`
- Output templates and report structures go in `assets/`
- Prefer `uk-property-mcp` over `property-shared` unless a specific tool only exists in the latter
- Accept Rightmove URLs as valid input for uk-property skills — extract listing ID from the URL pattern `rightmove.co.uk/properties/{id}`
- Version skills in frontmatter (`version: "1.0"`); increment on any structural change to the workflow
