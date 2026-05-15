# Celestine Skills

Tracks built and planned skills using the Celestine MCP (Swiss Ephemeris astrological calculations).

---

## Built

_None yet._

---

## Planned / Ideas

### `natal-reading`
Full birth chart interpretation workflow.
Tools: `birth_chart` (detail=full) → resource reads for planets/signs/houses/aspects
Output: structured natal chart narrative covering chart shape, key planets, house themes.

### `transit-forecast`
Transits over a user-specified date range with narrative interpretation.
Tools: `transit_search` → `current_transits` → resource reads for relevant aspects
Output: ranked significant transits, themes for the period, key dates to watch.

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
