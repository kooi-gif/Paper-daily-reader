---
name: paper-daily-reader
description: Daily arXiv/OpenReview candidate fetch, semantic scoring, and Top-N academic day report. Use when the user asks for 今日论文, 每日论文, arXiv 日报, paper daily, candidate pool, Top 10 papers, OpenReview candidates, or wants to score/filter recent papers by research profile. Complementary to embodied-paper-assistant (GitHub watchlist + PDF deep analysis)—do not use this skill to track named GitHub repos; do not run broad keyword scraping outside this daily pipeline.
compatibility: OpenCode 1.18+; Python 3.11+; conda recommended
metadata:
  audience: researchers
  workflow: daily-paper-discovery
---

# Paper Daily Reader

Turn candidate retrieval into an OpenCode workflow: **fetch → semantic score → Top-N day report**. Scripts only build the candidate pool; the agent reads and writes the report.

## Non-negotiable rules

- **Scripts never write the final report.** After fetch, the agent must read candidates and score them.
- Never invent a paper, score, metric, or PDF claim. Use `未知` when evidence is missing.
- Label every summary's evidence: `abstract-only` / `paper page` / `pdf`.
- If `duplicate_check.status == duplicate_of_previous` or `recommended_action == write_no_new_batch_note`, write a short **无新候选批次** note only—do not reuse yesterday's Top N.
- If network preflight fails, stop and report the error. Do not write an empty report.
- Default Top N is **10** (user may override). Batch depth (deep-read count) default ≈15.
- Keep runtime artifacts under `<workspace>/workspace/paper-daily/`, never inside the skill directory.
- Do not `git push`. Do not silently change the research profile—confirm with the user first.
- `keyword_score` / `coarse_retrieval_score` are retrieval hints only, never final recommendations.

## Runtime layout

```text
SKILL_DIR = directory containing this SKILL.md
PYTHON    = python from conda env paper-assistant-daily (see environment.yml)

<workspace>/
  workspace/paper-daily/
    config.yaml
    data/raw/YYYY-MM-DD.json
    data/processed/YYYY-MM-DD_candidates.json
    data/processed/YYYY-MM-DD_scored.json
    logs/YYYY-MM-DD.log
    reports/YYYY-MM-DD.md
    research_profile.md          # optional private notes
```

First run: ensure `workspace/paper-daily/config.yaml` exists (copy from `vendor/paper-daily/config.yaml` if missing).

### Run scripts correctly

Set `PYTHONPATH` (or cwd) to the scripts folder so `from utils import ...` works:

```powershell
$scripts = Join-Path $SKILL_DIR "vendor/paper-daily/scripts"
$env:PYTHONPATH = $scripts
# optional proxy if arXiv/OpenReview blocked:
# $env:PAPER_DAILY_USE_ENV_PROXY = "1"

& $PYTHON (Join-Path $scripts "daily_papers.py") `
  --config (Resolve-Path "workspace/paper-daily/config.yaml") `
  --date today --stage fetch --force
```

## Route the request

| User intent | Read |
|---|---|
| Fetch today's candidates, check network, handle duplicate/empty batch | `references/fetch.md` |
| Score candidates, deep-read, write Top-N day report | `references/score_report.md` |
| Edit research profile, keywords, sources, Top N default, output dirs | `references/config.md` |

Combined “跑今日日报” = fetch then score_report. Load `config.md` only when the user changes settings.

## Standard workflow

```text
ensure workspace/paper-daily/config.yaml
  → daily_papers.py --stage fetch
  → read candidates + raw recommended_action
  → (duplicate / empty → short no-new-batch note → stop)
  → semantic score all candidates → scored.json
  → deep-read top ~15 (PDF/page when possible)
  → write reports/YYYY-MM-DD.md (Top N)
  → reply: counts, titles, paths, evidence limits
```

## First-time environment

```powershell
conda env create -f "$SKILL_DIR/environment.yml"
conda activate paper-assistant-daily
```

If the env exists: `conda env update -n paper-assistant-daily -f "$SKILL_DIR/environment.yml" --prune`.

## Relationship to embodied-paper-assistant

| Skill | Use for |
|---|---|
| `paper-daily-reader` | Broad **daily discovery** from arXiv/OpenReview by research profile |
| `embodied-paper-assistant` | **Named GitHub repos**, download selected PDFs, architecture briefs, deep analysis |

Handoff: if the user wants deep architecture analysis of a paper from the daily report, switch to `embodied-paper-assistant` (download/extract/analyze) after the user selects IDs.

Thank you to arXiv and OpenReview for open access APIs.
