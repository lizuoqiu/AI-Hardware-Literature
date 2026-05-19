# AI x Hardware in HCI Literature Review

This repository contains a reproducible pipeline for a systematic literature review of AI combined with hardware, broadly defined, in top HCI venues from the last 10 years.

## Interactive Literature Maps

The public-facing dashboard files live in `docs/` so this repository can be served with GitHub Pages.

- Citation and continuity dashboard: `docs/citation_continuity_dashboard.html`
- Paper reading matrix: `docs/reading_matrix.html`

Published GitHub Pages URL:

`https://lizuoqiu.github.io/AI-Hardware-Literature/`

Direct links:

- Citation and continuity dashboard: `https://lizuoqiu.github.io/AI-Hardware-Literature/citation_continuity_dashboard.html`
- Paper reading matrix: `https://lizuoqiu.github.io/AI-Hardware-Literature/reading_matrix.html`

Default scope for this run:

- Date range: 2016-01-01 through 2026-05-13.
- Venues: CHI full/main proceedings, UIST main proceedings, IMWUT, TOCHI, and CSCW / PACM HCI CSCW-related archival articles.
- Inclusion rule: a paper must be both AI/ML/intelligent-model related and hardware, sensor, actuator, robot, XR, mobile/ubiquitous, fabrication, assistive-device, or tangible-interface related.

The pipeline saves intermediate JSONL checkpoints and final CSV/SQLite outputs so the review can be resumed and audited.

## Project Layout

```text
.
├── scripts/
│   ├── 01_collect_venue_papers.py
│   ├── 02_screen_ai_candidates.py
│   ├── 03_screen_hardware_candidates.py
│   ├── 04_fetch_full_text_or_metadata.py
│   ├── 05_read_and_extract_paper_fields.py
│   ├── 06_build_citation_graph.py
│   ├── 07_author_analysis.py
│   ├── 08_method_ai_hardware_contribution_stats.py
│   ├── 09_continuity_analysis.py
│   ├── 10_generate_visualizations.py
│   ├── 11_generate_report.py
│   ├── 12_sync_to_notion.py
│   ├── litreview_common.py
│   └── run_pilot.py
├── data/
│   ├── raw/
│   ├── intermediate/
│   └── final/
├── prompts/
└── tests/
```

## Install

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
```

The collection and screening scripts use public metadata APIs. They include caching, rate limiting, and retry/backoff. They do not fetch pirated PDFs.

## Run A Small Pilot

The default pilot uses UIST 2024, fetches DBLP and OpenAlex metadata, screens AI + hardware candidates, extracts 5-10 structured records, and generates a pilot CSV.

```bash
python scripts/run_pilot.py --venue UIST --year 2024 --max-records 10
```

Outputs:

- `data/intermediate/all_venue_papers.jsonl`
- `data/intermediate/metadata_enriched.jsonl`
- `data/intermediate/ai_candidates.jsonl`
- `data/intermediate/hardware_candidates.jsonl`
- `data/intermediate/included_papers.jsonl`
- `data/intermediate/excluded_candidates.jsonl`
- `data/intermediate/paper_reading_notes.jsonl`
- `data/final/pilot_papers.csv`
- `data/final/ai_hardware_hci_litreview.sqlite`

## Run Full Corpus

After the pilot is reviewed:

```bash
python scripts/01_collect_venue_papers.py --start-year 2016 --end-year 2026
python scripts/04_fetch_full_text_or_metadata.py --input data/intermediate/all_venue_papers.jsonl
python scripts/02_screen_ai_candidates.py --input data/intermediate/metadata_enriched.jsonl
python scripts/03_screen_hardware_candidates.py --input data/intermediate/ai_candidates.jsonl
python scripts/05_read_and_extract_paper_fields.py --input data/intermediate/hardware_candidates.jsonl
python scripts/06_build_citation_graph.py
python scripts/07_author_analysis.py
python scripts/08_method_ai_hardware_contribution_stats.py
python scripts/09_continuity_analysis.py
python scripts/10_generate_visualizations.py
python scripts/11_generate_report.py
python scripts/12_sync_to_notion.py
python scripts/13_quality_control.py
```

## Notion Sync

The local scripts look for:

- `NOTION_TOKEN`
- `NOTION_PARENT_PAGE_ID`

If either is missing, `scripts/12_sync_to_notion.py` writes Markdown/JSON payloads for manual import instead of stopping the review. In this Codex session, the Notion plugin was also tested directly against the supplied parent page:

`35f91154-fbc6-804f-966a-d626fe1c8bd3`

## Reproducibility

All statistics are computed from structured files or SQLite tables. Do not edit final statistics manually. If a paper lacks abstract/full-text evidence, the extractor marks it as `metadata_only`, `abstract_only`, or low confidence.
