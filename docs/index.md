# eng-metrics-suite

A self-hosted engineering metrics pipeline: import commit and PR/MR stats
from your GitHub, GitLab, Bitbucket, or Azure DevOps org into Postgres,
and generate a PDF report (team/author activity, PR review health, cycle
time, repo trends) from it.

The [eng-metrics-suite](https://github.com/jsooter/eng-metrics-suite) repo
is just the `docker-compose.yml` needed to run it — the application
images (`git-processor`, `pr-processor`, `eng-reports`) are published
pre-built at [ghcr.io/jsooter](https://github.com/jsooter?tab=packages).
Free to use; see [LICENSE](https://github.com/jsooter/eng-metrics-suite/blob/master/LICENSE).

## Requirements

- Docker + Docker Compose

## The pieces

- **`git-processor`** — clones queued repos and imports commit stats.
- **`pr-processor`** — imports PR/MR stats from the same repos' vendor APIs.
- **`eng-reports`** — reads both, renders a PDF report on demand.
- **Postgres** — shared database all three read/write.

All three application services log structured JSON to stdout and are
controlled by the same `docker compose` commands — see
[Getting Started](getting-started.md) to begin.
