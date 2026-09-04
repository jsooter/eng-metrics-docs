# Generating Reports

```
docker compose run --rm eng-reports report.py --output /out/report.pdf
```

If your host user isn't UID 1000 (check with `id -u`), add `--user
"$(id -u):$(id -g)"` so the output file comes out owned by you instead of
UID 1000:

```
docker compose run --rm --user "$(id -u):$(id -g)" eng-reports report.py --output /out/report.pdf
```

The PDF lands in `./reports/report.pdf` on the host (that directory is
bind-mounted into the container).

## Useful flags

- `--period last-week|last-month|last-quarter` or `--start YYYY-MM-DD --end YYYY-MM-DD`
  (default: last 30 days)
- `--repo identity_key` (repeatable) and/or `--org github.com/owner` to
  scope the report instead of covering every repo you've imported
- `--team-map /var/lib/eng-metrics-suite/teams.csv` to roll up commit
  activity by team instead of by individual author

## Team rollups

Put a CSV file with an `email,team` header at
`/var/lib/eng-metrics-suite/teams.csv` on the host — CSV rather than a
config-file format like YAML/JSON, so whoever maintains the roster (often
not an engineer) can keep it in a spreadsheet and export/save as CSV:

```csv
email,team
alice@example.com,Platform
bob@example.com,Platform
carol@example.com,Product
```

Anyone not listed shows up under "Unmapped" in the report rather than
being silently dropped.

If the same person shows up as multiple rows in the author table, or
lands in "Unmapped" despite being in your team-map CSV, see
[Author Identity Consistency](author-identity.md).

## Investment allocation report

A second, independent script in the same `eng-reports` image: shows
how work breaks down by category (feature/bug/tech-debt/etc.), sourced
from Jira ticket data via the optional
[issue-processor](https://github.com/GitUltraHQ/issue-processor)
allocation-tracking flow (separate from the change-failure-rate/MTTR
Jira integration -- see that repo's README to configure either or
both).

```
docker compose run --rm eng-reports allocation_report.py --output /out/allocation.pdf
```

Same `--period`/`--start`/`--end`/`--output` flags as `report.py`
above, but **no `--repo`/`--org`/`--team-map` scoping** -- it always
covers every tracked Jira project as one combined report (Jira tickets
have no repo relationship the way commits/PRs do, so there's nothing
to scope by). If the allocation flow isn't configured at all, the PDF
renders a single "not configured" page rather than misleading empty
tables.

Next: [Scheduling Reports](scheduling.md) to get this running automatically.
