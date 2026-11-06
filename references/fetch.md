# Fetch module

Load when the user wants candidates only, or as step 1 of a full daily report.

## Preconditions

1. `workspace/paper-daily/config.yaml` exists (copy from `vendor/paper-daily/config.yaml` on first use; then edit research profile).
2. Conda env `paper-assistant-daily` active (or PYTHONPATH + deps installed).
3. User did **not** ask to change sources/profile in the same breath—otherwise read `config.md` first.

## Command

```powershell
$scripts = Join-Path $SKILL_DIR "vendor/paper-daily/scripts"
$env:PYTHONPATH = $scripts
# If preflight fails and the sandbox injects a broken proxy:
# $env:PAPER_DAILY_USE_ENV_PROXY = "1"

& $PYTHON (Join-Path $scripts "daily_papers.py") `
  --config (Resolve-Path "workspace/paper-daily/config.yaml") `
  --date today --stage fetch --force
```

Useful flags:

| Flag | Meaning |
|---|---|
| `--date 2026-11-06` | Historical date instead of `today` |
| `--lookback-days 7` | Widen source lookback |
| `--sources arxiv` | Only arxiv (or `arxiv,openreview`) |
| `--force` | Overwrite existing candidates for that date |

Exit code `2` = network preflight or fetch-stage error. Report stderr; do not invent results.

## Outputs to read

| File | Purpose |
|---|---|
| `workspace/paper-daily/data/processed/YYYY-MM-DD_candidates.json` | Candidate pool for scoring |
| `workspace/paper-daily/data/raw/YYYY-MM-DD.json` | `duplicate_check`, `recommended_action` |
| `workspace/paper-daily/logs/YYYY-MM-DD.log` | Failure diagnosis |

Parse the script stdout for the exact paths printed (`Candidates:`, `Raw:`).

## Branching after fetch

1. **Network preflight failed** → stop; report URL/error; suggest proxy env or retry; no report file.
2. **`recommended_action == write_no_new_batch_note`** OR **`duplicate_check.status == duplicate_of_previous`** OR candidates empty OR same ID set as previous day → write short note to `reports/YYYY-MM-DD.md`:
   - candidate count
   - recent visible arXiv batch date if known
   - reason (no announcement batch / duplicate pool / etc.)
   - do **not** rewrite `*_scored.json` unless the user asks to re-score
3. **Candidates present** → proceed to `references/score_report.md`.

## Failure checklist

- Read `logs/YYYY-MM-DD.log`, raw JSON, candidates JSON.
- Distinguish: proxy / TLS / rate limit / empty announcement day.
- Only with explicit user OK may you fall back to a previous non-empty candidates file—and label the source date at the top of the report.
