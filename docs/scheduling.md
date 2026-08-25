# Scheduling Reports

`docker compose run` is a normal one-shot command — wire it into a host
cron job or systemd timer to get a report on a recurring schedule (e.g.
weekly, for a Monday-morning exec digest):

```
0 8 * * 1 cd /path/to/eng-metrics-suite && docker compose run --rm eng-reports report.py --period last-week --output /out/weekly-report.pdf
```

Adjust the cron expression, `--period`, and output filename to taste —
nothing about scheduling is special-cased in `eng-reports` itself, it's
just a script that takes `--output` and exits.
