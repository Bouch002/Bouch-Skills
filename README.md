# Bouch Skills

Agent Skills for [bouch.dev](https://bouch.dev) — reusable, structured workflows built on top of the Bouch MCP servers, conforming to the [agentskills.io](https://agentskills.io) specification.

Each skill is a folder containing a `SKILL.md` that an AI agent can load and follow. Skills wrap one or more MCP tool chains into a consistent, tested workflow with a defined output format.

---

## What's in this repo

```
Bouch-Skills/
├── uk-property/          # Skills using uk-property-mcp
├── celestine/            # Skills using the Celestine astrology MCP
├── uk-archives/          # Skills using UK archival data (MCP TBC)
└── docs/                 # Spec reference and supporting docs
```

Each domain folder contains:

- `BUILDS.md` — log of built skills and backlog of planned ones
- `<skill-name>/SKILL.md` — the skill itself
- `<skill-name>/references/` — interpretation guides, benchmarks, lookup tables
- `<skill-name>/assets/` — output templates and static resources

---

## Available skills

### uk-property

| Skill | Description | Status |
|---|---|---|
| [`property-investment-report`](uk-property/property-investment-report/SKILL.md) | Full BTL investment analysis from a Rightmove URL or address. Covers yield, rental market, EPC, SDLT, cash and mortgage scenarios, with auto deep-dive on positive signals. | ✅ v2.2 |

**Planned:** `property-block-analysis`, `planning-research`, `company-profile`, `listing-analysis`

### celestine

**Planned:** `natal-reading`, `transit-forecast`, `lunar-calendar`, `progression-report`, `synastry-snapshot`

### uk-archives

On hold — data source TBC.

---

## Prerequisites

To run these skills you need:

1. **Claude Code** (or any Claude agent that supports the agentskills.io skill format)
2. **Bouch MCP servers** connected to your Claude environment:
   - `uk-property-mcp` — for all uk-property skills
   - `Celestine-mcp` — for all celestine skills
3. Access to [bouch.dev](https://bouch.dev) for MCP server credentials

If you're a bouch.dev collaborator, the MCP servers should already be configured in your Claude setup. Check with the team if you're missing a connection.

---

## Using a skill

Point Claude at the skill folder or paste the `SKILL.md` contents. The skill will guide Claude through the workflow automatically.

**Quickest way — Rightmove URL:**

```
Run the property-investment-report skill for:
https://www.rightmove.co.uk/properties/12345678
```

**With full address:**

```
Run the property-investment-report skill for:
42 High Street, Derby, DE1 2AB — purchase price £250,000, cash buyer
```

Claude will follow the skill steps, call the MCP tools, ask for any missing data it needs, and produce a structured report with traffic light confidence ratings on each section.

---

## Understanding report confidence ratings

All reports produced by these skills use a traffic light system to show how well each section's data has been verified:

| Rating | Meaning |
|---|---|
| 🟢 Confirmed | Data sourced from MCP tools and verified to match the subject property |
| 🟡 Estimated | Approximate, area-level, or from a proxy — see the 📋 re-run note |
| 🔴 Unconfirmed | Missing or known to be from a different property — due diligence required |

Where data is 🟡 or 🔴, the report includes a `📋 Re-run with [x] to confirm` note explaining exactly what's needed to improve that section.

---

## Building a new skill

1. Check the relevant `BUILDS.md` for the domain — pick something from the Planned list or add a new idea
2. Create a folder: `<domain>/<skill-name>/` (lowercase, hyphens only)
3. Write `SKILL.md` using the template in `CLAUDE.md`
4. Add `references/` files for any interpretation guidance or benchmarks
5. Test the skill against real data before marking as built
6. Move the entry from **Planned → Built** in `BUILDS.md` with the ship date

Full conventions and workflow patterns are documented in [`CLAUDE.md`](CLAUDE.md).

### Key rules

- Skill folder name must match the `name` field in `SKILL.md` frontmatter exactly
- `description` must be keyword-rich — agents use it to decide when to activate the skill
- Keep `SKILL.md` under 500 lines — push detail to `references/`
- List all MCP tools the skill will call in `allowed-tools`
- Follow the **listing-first** pattern for uk-property skills (read the Rightmove listing before running financial tools)
- All investment skills must show both **cash** and **BTL mortgage** scenarios
- Apply traffic light ratings to every section of every report

---

## Contributing

This is an internal bouch.dev project. If you're a collaborator:

- Branch from `master`, build your skill, open a PR
- Test against at least two real properties before marking a skill as built
- Update `BUILDS.md` when you ship
- Keep `CLAUDE.md` up to date with any new workflow patterns you establish — future Claude sessions will read it

Questions or ideas → speak to the team at [bouch.dev](https://bouch.dev).
