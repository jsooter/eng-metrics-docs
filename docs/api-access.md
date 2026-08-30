# API Access (Team / Enterprise)

Everything on this site so far covers the free, self-hosted pipeline:
import commit/PR data, then generate a PDF report on demand. **Team and
Enterprise plans** add a REST/JSON API on top of the same database, so
your own internal dashboards (Grafana, an internal tool, whatever you
already use) can pull these metrics directly and on your own schedule,
instead of regenerating and re-reading a PDF.

Like the rest of this pipeline, it's **self-hosted**: it runs in your
own infrastructure, against your own database, authenticated with a key
you generate and control yourself. No data leaves your instance to make
this work, and there's nothing to sign up for beyond the plan itself.

## Endpoints

All endpoints return JSON and accept a required `start`/`end` date
range, plus optional scoping (see below).

| Endpoint | What it returns |
|---|---|
| Author activity | Per-author commits, lines added/removed, files changed, churn ratio, and active-days/week (a consistency-of-contribution signal) -- plus PR counts and average commits per PR. |
| Review health | PR/merge counts, cycle time broken into pickup (open → first review) and review (first review → merge) stages with p50/p90 percentiles, per-reviewer load, and a review-load-balance (Gini coefficient) score. |
| Repo trends | Weekly commit/PR volume and churn ratio, per repo. |
| Cycle time | The pickup/review/total percentile breakdown as its own endpoint, for dashboards that only need that slice. |
| Deployment frequency | Weekly deployment counts per repo (every git tag counts by default; scope down to a real deploy-tagging convention with a tag-pattern filter). |
| Lead time for changes | p50/p90/avg hours from a commit to its nearest later tag -- one of the four DORA metrics. |
| AI usage | Per-author and org-wide weekly trend of commits with an AI-tool `Co-authored-by:` trailer (the same convention GitHub itself reads to show a tool's avatar on a commit) -- a lower bound on AI-assisted work, not a precise measurement. |

## Scoping

Every endpoint accepts:

- `repo` (repeatable) and/or `org` -- limit to specific repos, same as
  the free CLI's `--repo`/`--org`.
- `team` -- limit to a team's repos and (where the data has a real git
  author identity, like commits and AI usage) filter to that team's own
  members, not just repos they happen to touch. Requires a team roster
  to be configured. PR/review data and deployment frequency can't be
  attributed to a team the same way (a PR author is a vendor-API
  username, not a git email; a tag isn't authored by anyone), so those
  stay repo-scoped.

`repo`/`org` and `team` are mutually exclusive.

## Example

```
curl "https://your-instance/v1/author-activity?start=2026-01-01&end=2026-04-01&team=Platform" \
    -H "Authorization: Bearer <your key>"
```

```json
{
  "commit_activity": [
    {
      "author_email": "alice@example.com",
      "author_name": "Alice Anderson",
      "commits": 47,
      "added": 1704,
      "removed": 340,
      "churn_ratio": 0.17,
      "active_days_per_week": 1.6
    }
  ],
  "meta": {
    "start": "2026-01-01T00:00:00Z",
    "end": "2026-04-01T00:00:00Z",
    "team_scope": {"team": "Platform", "repo_count": 20}
  }
}
```

## Auth

Every request needs `Authorization: Bearer <key>`. The key isn't issued
or stored by us -- you generate it yourself and set it in your own
deployment config, the same way you already set your own database
password. Once you have plan access, there's no separate account/key
request step.

### Generating a key

Any sufficiently random string works -- a UUID, a password manager's
generator, whatever you already reach for. If you don't have a
preferred method:

```
openssl rand -hex 32
```

or, without `openssl`:

```
python3 -c "import secrets; print(secrets.token_hex(32))"
```

Set the result as `ENG_API_KEY` in your `.env` (if you're running the
`eng-metrics-suite-pro` compose bundle) or as the `API_KEY` environment
variable directly (if running the container standalone) -- then send it
back as `Authorization: Bearer <that value>` on every request. Treat it
like any other credential: don't commit it, and rotate it (just
generate a new one and restart the container) if you suspect it's
leaked.

## Getting access

Available on **Team** and **Enterprise** plans -- see
[gitultra.com](https://gitultra.com) for plan details, or apply for
beta access there if you're interested.
