# Running Workers

`git-processor` and `pr-processor` (already running from `docker compose
up`) pick up queued repos automatically and import commit/PR stats into
Postgres. `issue-processor` does the same for Jira data if you've
configured it (see [Jira Integration](jira-integration.md)). Watch
progress with:

```
docker compose logs -f git-processor pr-processor issue-processor
```

See [Logs](logs.md) for what each JSON line means.

## Scaling workers

There's no worker-count setting — `git-processor` and `pr-processor` each
claim one repo at a time from the shared queue (`FOR UPDATE SKIP LOCKED`,
so concurrent claims never collide), and by default `docker compose up`
runs exactly one of each. To process more repos in parallel, run more
instances with Compose's `--scale`:

```
docker compose up -d --scale git-processor=4 --scale pr-processor=4
```

Each replica still exits once the queue's empty and gets restarted
independently by `restart: unless-stopped`, so the scale factor persists
without further action. `--max-attempts`/`--lease-minutes` are per-repo,
not per-worker, so scaling up doesn't change retry behavior — it just
means more repos get claimed per pass.

!!! warning "Bitbucket Cloud and scaling"
    Be more cautious scaling `pr-processor` if you're importing from
    **Bitbucket Cloud**: it's known to temporarily block source IPs that
    get hit too aggressively. `pr-processor` throttles and backs off
    automatically, but that protection is per-worker-process, not
    coordinated across replicas — N scaled workers still add up to N× the
    request rate from this host's IP. GitHub/GitLab's limits are generous
    enough that this isn't a practical concern there.

Next: [Generating Reports](generating-reports.md) once some data's imported.
