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

## Conventions
- Place all interpretation guidance (e.g. EPC rating scales, yield benchmarks, astrological orbs) in `references/` not in `SKILL.md`
- Output templates and report structures go in `assets/`
- Prefer `uk-property-mcp` over `property-shared` unless a specific tool only exists in the latter
