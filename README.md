# Outside Baseball — Analytics

A collection of Jupyter notebooks that compute custom baseball analytics metrics. Each notebook defines a metric, pulls pitch-by-pitch data from Baseball Savant, and produces normalized per-pitcher scores across MLB seasons.

Results are published on [Outside Baseball](https://outside-baseball.web.app).

By [Aryan Kapoor](https://aryankapoor.web.app).

---

## Projects

| Metric | Notebook | Seasons |
|--------|----------|---------|
| [Composure+](https://outside-baseball.web.app/composure) | `composure/composure.ipynb` | 2021–2025 |

---

## Repository Structure

```
baseball/
├── projects.json           ← S3 sync manifest (one entry per project)
├── composure/
│   ├── composure.ipynb     ← metric definition + full pipeline
│   ├── data/               ← raw parquet cache (not synced to S3)
│   │   ├── all_pitches_YYYY.parquet
│   │   └── pitcher_metrics_YYYY.parquet
│   └── results/            ← final scored CSVs (synced to S3)
│       └── composure_scores_YYYY.csv
└── .github/workflows/
    └── sync-to-s3.yml      ← auto-deploys results to S3 on push
```

Each metric follows the same 5-phase pipeline: config → data pull → metric computation → scoring → analysis.

---

## Setup

```bash
pip install pybaseball requests pandas numpy matplotlib tqdm pyarrow
```

Data is fetched from Baseball Savant via `pybaseball`. Enable disk caching before any fetches to avoid re-downloading game data:

```python
import pybaseball
pybaseball.cache.enable()
```

---

## Running a Notebook

Open the notebook for the metric you want to run and execute phases top to bottom. Each phase saves its output so you can re-run later phases independently without re-fetching data.

| Phase | What it does | Re-run when |
|-------|-------------|-------------|
| 0 — Config | Set thresholds, weights, normalization | Any parameter change |
| 1 — Data pull | Fetch pitch-by-pitch from Baseball Savant | New season or first run |
| 2 — Metric computation | Compute raw per-pitcher rates | Threshold changes |
| 3 — Scoring | Normalize, weight, output CSVs | Weight or normalization changes |
| 4 — Analysis | Validation and diagnostics | After any results change |

---

## S3 Deployment

Pushing to `main` automatically syncs results to AWS S3 via GitHub Actions. The workflow reads `projects.json` — no workflow edits needed when adding a new project.

Each project syncs to its own S3 bucket (`outside-baseball-{slug}`):

```
outside-baseball-{slug}/
  notebooks/{slug}.ipynb     ← notebook for the web app to display
  data/{results CSVs}        ← scored output files
  seasons.json               ← auto-generated list of available years
```

### Adding a new project

1. Create a folder following the standard layout
2. Add an entry to `projects.json`
3. Create the S3 bucket (one-time, manual — see `CLAUDE.md` for the exact commands)
4. Push to `main` — the workflow handles the rest
5. Add the project entry to the web repo's `src/data/projects.js`
