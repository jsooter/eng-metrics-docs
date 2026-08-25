# eng-metrics-docs

Source for the [eng-metrics-suite](https://github.com/jsooter/eng-metrics-suite)
documentation site, built with [MkDocs](https://www.mkdocs.org/) +
[Material](https://squidfunk.github.io/mkdocs-material/).

**Live site: [jsooter.github.io/eng-metrics-docs](https://jsooter.github.io/eng-metrics-docs/)**

Deploys automatically to GitHub Pages on every push to `master`.

## Editing locally

```
python3 -m venv .venv
.venv/bin/pip install mkdocs-material
.venv/bin/mkdocs serve
```

Then edit anything under `docs/` — pages are listed in `mkdocs.yml`'s `nav`.
