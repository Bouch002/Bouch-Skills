# Celestine Skills

Tracks built and planned skills using the Celestine MCP (Swiss Ephemeris astrological calculations).

---

## Built

### `natal-reading` — 2026-05-17
Full birth chart interpretation workflow. Three phases: calculate chart via `birth_chart`, enrich key placements with sign/planet/house/aspect archetypes, synthesise into a structured reading covering the Big Three, personal planets, chart ruler, major aspects, and house themes. Gracefully degrades when birth time is unknown.

### `transit-forecast` — 2026-05-18
Planetary transit forecast over a user-specified date range. Three phases: calculate transits via `transit_search` (+ `current_transits` if today falls in range), enrich top 8 transits with planet and aspect archetypes, synthesise into a structured forecast with ranked transits, key dates, and period themes. Handles multi-hit retrograde transits, station intensification, and gracefully degrades when birth time is unknown.

---

## Planned / Ideas

### `lunar-calendar`
Monthly lunar and astrological calendar with downloadable .ics.
Tools: `generate_calendar_events` → surface `download_url`
Output: calendar of moon phases, ingresses, key transits + .ics link.

### `progression-report`
Secondary progressions + solar arc report for a birth chart.
Tools: `progression_report` → resource reads for progressed planets
Output: current progressed chart themes, active solar arc directions.

### `synastry-snapshot`
Quick compatibility overview between two birth charts.
Tools: `birth_chart` × 2 → aspect comparisons via `get_aspect`
Output: key inter-chart aspects and their themes.
