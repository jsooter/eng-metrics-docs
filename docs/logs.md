# Logs

All three application services log structured JSON lines to stdout — one
JSON object per line, with `timestamp`, `level`, `component`, `message`,
plus whatever fields are relevant to that specific event (e.g.
`identity_key`, `imported`, `failed`).

```
docker compose logs -f git-processor pr-processor
```

## Verbosity

Controlled by `LOG_LEVEL` in `.env` — `INFO` (default), `DEBUG`,
`WARNING`, or `ERROR`.

## Failures

Failures include a full traceback under an `"exception"` field, not just
a one-line message — pipe through `jq` if you want to pull just that
field out. `docker compose logs` prefixes each line with the container
name by default, so add `--no-log-prefix` to get pure JSON lines `jq` can
parse:

```
docker compose logs --no-log-prefix pr-processor | jq -r 'select(.level=="ERROR") | .exception'
```
