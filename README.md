# Paper Daily Reader

OpenCode skill for daily arXiv/OpenReview candidate retrieval, semantic scoring, and Top-N day reports.

Split of responsibilities:

| Layer | Does | Does not |
|---|---|---|
| `vendor/paper-daily` scripts | Fetch candidates (arXiv / OpenReview / optional OpenAlex), normalize, dedupe, coarse-rank | Write the final Top-10 report |
| OpenCode agent (this skill) | Semantic scoring, optional deep read, daily Markdown report | Invent papers or metrics |

Runtime data lives under the **workspace**, never inside this skill directory:

```text
workspace/paper-daily/
  config.yaml     # copy of research profile + sources (edit me)
  data/raw/
  data/processed/
  logs/
  reports/
```

## Commands

See `SKILL.md` for the full agent workflow. Typical shell (from workspace root):

```powershell
# ensure runtime config exists
# copy vendor config -> workspace/paper-daily/config.yaml (first run)

python "$SKILL_DIR/vendor/paper-daily/scripts/daily_papers.py" `
  --config "workspace/paper-daily/config.yaml" `
  --date today --stage fetch --force
```

Run scripts with `cwd` = `vendor/paper-daily/scripts` (or set `PYTHONPATH` to that folder) so local imports resolve.

## Tests

```powershell
cd "$SKILL_DIR/vendor/paper-daily"
python -m pytest tests
```

## Uninstall

Delete only `.opencode/skills/paper-daily-reader/`. Never delete `workspace/paper-daily/` as part of uninstall.
