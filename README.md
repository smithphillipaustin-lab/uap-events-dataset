# UAP Events Dataset

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Refreshed nightly](https://img.shields.io/badge/Refreshed-nightly-blue.svg)](https://github.com/smithphillipaustin-lab/uap-events-dataset/actions)
[![Live source](https://img.shields.io/badge/Live%20source-disclosurearchives.com-a4441e.svg)](https://disclosurearchives.com)

A machine-readable, citation-grade dataset of every documented Unidentified Anomalous Phenomena (UAP) event in the public record: U.S. and international government hearings, declassified documents, official reports, named witness testimony, and major sightings — each entry dated and linked to a primary source.

The canonical, human-readable version of the archive lives at **[disclosurearchives.com](https://disclosurearchives.com)**. This repo is the same data in CSV and JSON, refreshed nightly from the live site, so you can drop it into a notebook, vector DB, RAG pipeline, dashboard, or research project without scraping HTML.

## Files

| File | Format | Use for |
|---|---|---|
| [`uap-events.csv`](./uap-events.csv) | CSV | Spreadsheets, R, pandas, SQL imports |
| [`uap-events.json`](./uap-events.json) | JSON | Notebooks, JS apps, RAG, vector DBs |

Both files share the same underlying records — the JSON form keeps `tags` and `witnesses` as proper arrays; the CSV uses pipe-delimited (`|`) strings for those columns to stay single-row-per-event.

## Quick start

```python
# pandas
import pandas as pd
df = pd.read_csv("https://raw.githubusercontent.com/smithphillipaustin-lab/uap-events-dataset/main/uap-events.csv")
df[df["country"] == "Brazil"].sort_values("event_date", ascending=False).head()
```

```r
# R
library(readr); library(dplyr)
events <- read_csv("https://raw.githubusercontent.com/smithphillipaustin-lab/uap-events-dataset/main/uap-events.csv")
events |> filter(event_type == "hearing") |> arrange(desc(event_date))
```

```bash
# jq
curl -s https://raw.githubusercontent.com/smithphillipaustin-lab/uap-events-dataset/main/uap-events.json \
  | jq '.events[] | select(.eventType=="hearing") | {title, eventDate, canonical}'
```

```js
// Node / browser
const res = await fetch("https://raw.githubusercontent.com/smithphillipaustin-lab/uap-events-dataset/main/uap-events.json");
const { events } = await res.json();
const navyEncounters = events.filter(e => e.tags.some(t => t.slug === "navy"));
```

For sub-day freshness or per-event lookups, prefer the live JSON API at [disclosurearchives.com/api/v1/events](https://disclosurearchives.com/api/v1/events) — same data, paginated, with `since`/`country`/`type` filters and CORS open.

## Schema

| Column (CSV) | Field (JSON) | Type | Notes |
|---|---|---|---|
| `slug` | `slug` | string | Stable URL slug and primary key. |
| `title` | `title` | string | Editorial title. |
| `event_date` | `eventDate` | ISO date | `YYYY-MM-DD`. Falls back to month-start when only month is known. |
| `date_display` | `dateDisplay` | string | Editorial date label (e.g. "April 2004", "Spring 1976"). |
| `event_type` | `eventType` | enum | `hearing`, `document_release`, `report`, `sighting`, `witness_testimony`, `official_statement`. |
| `country`, `region`, `location_name` | `location.{country,region,name}` | string | As stored in the archive. Empty if the event has no geographic anchor. |
| `lat`, `lng` | `location.{lat,lng}` | float | WGS84. |
| `source_name` | `sourceName` | string | Primary source publisher (e.g. "AARO", "ODNI", "NASA UAP Office"). |
| `source_url` | `sourceUrl` | string | Direct link to the primary source. |
| `summary` | `summary` | string | 2–4 sentence editorial summary. |
| `tags` | `tags[]` | pipe-delim / array | Topical tags. JSON form is `[{slug, name}]`. |
| `witnesses` | `witnesses[]` | pipe-delim / array | Named witnesses. JSON form is `[{slug, name}]`. |
| `canonical_url` | `canonical` | URL | The per-event page on disclosurearchives.com — use this when citing. |
| `updated_at` | `updatedAt` | ISO timestamp | Last editorial edit. |

## Refresh cadence

The CSV and JSON files are **regenerated nightly** by a GitHub Actions workflow that pulls the latest dumps from the live site ([`.github/workflows/refresh.yml`](.github/workflows/refresh.yml)). Watch the [Actions tab](https://github.com/smithphillipaustin-lab/uap-events-dataset/actions) for the most recent run. The `updatedAt` field on each record tells you the editorial freshness of that specific entry.

For datasets with a hard freshness requirement (live event publishing, sub-day cadence), use the JSON API instead.

## License

The editorial copy and dataset structure are licensed [**CC BY 4.0**](https://creativecommons.org/licenses/by/4.0/). You can use the data in any project — academic, journalistic, or commercial — provided you credit *Disclosure Archives* and (where feasible) link to the canonical event URL.

> Primary-source documents linked from each event (`source_url`) retain their original publisher's terms — typically U.S. or foreign government works, peer-reviewed papers, or news articles. Treat those individually.

### Recommended citation

```
Disclosure Archives. UAP Event Dataset (CSV/JSON). disclosurearchives.com/data.
Mirror: github.com/smithphillipaustin-lab/uap-events-dataset. Retrieved YYYY-MM-DD.
```

## Editorial standards

Every event entry on the live site is linked to a primary government, military, or peer-reviewed source. The editorial standards, corrections policy, and funding disclosure are documented at [disclosurearchives.com/about](https://disclosurearchives.com/about). A daily AI monitor over .gov UAP feeds drafts new entries; high-confidence drafts from trusted sources auto-publish, the rest pass through human review.

## Built something with this?

We'd love to feature it. Tag [@TheUAPRecord](https://x.com/TheUAPRecord) on X or email <tips@disclosurearchives.com>.

## Related projects

- **[disclosurearchives.com](https://disclosurearchives.com)** — The live, human-readable archive.
- **[/api/v1/events](https://disclosurearchives.com/api/v1/events)** — Paginated, filterable read-only JSON API.
- **[/embed/timeline](https://disclosurearchives.com/embed)** — Drop-in iframe widget for blogs, Substacks, articles.
- **[/llms.txt](https://disclosurearchives.com/llms.txt)** — AI-crawler manifest for LLM citations.

## Contributing

Editorial corrections and new event submissions go through the live site:

- Editorial tips → [tips@disclosurearchives.com](mailto:tips@disclosurearchives.com)
- Corrections → [corrections@disclosurearchives.com](mailto:corrections@disclosurearchives.com)
- Submission form → [disclosurearchives.com/about#submit](https://disclosurearchives.com/about#submit)

This repo is a **mirror**; PRs against the data files themselves will be closed (the source-of-truth is the live database). PRs against the README, workflow, or schema documentation are welcome.
