# Config module

Load only when the user changes research profile, keywords, sources, output paths, or defaults.

## Files

| Path | Role |
|---|---|
| `workspace/paper-daily/config.yaml` | **Runtime** config actually passed to `--config` |
| `$SKILL_DIR/vendor/paper-daily/config.yaml` | Template shipped with the skill |
| `workspace/paper-daily/research_profile.md` | Optional free-form notes for scoring (not read by scripts) |

Never edit `workspace/papers/**` (that belongs to embodied-paper-assistant).

## Safe edit flow

1. Read current `workspace/paper-daily/config.yaml`.
2. Show the user a short diff intent (what keywords/venues/sources change).
3. Apply edits with the file editor (preserve YAML structure).
4. Do not run fetch unless the user asked to re-run after the change.

## Common changes

### Research focus

Edit `research_profile.goal`, `background_topics`, `positive_keywords`, `negative_keywords`.

Example embodied defaults are already in the shipped template (VLA / WAM / world models / manipulation).

### Sources

```yaml
sources:
  arxiv:
    enabled: true
    lookback_days: 3
  openreview:
    enabled: true
  openalex:
    enabled: false   # keep false unless user explicitly wants OpenAlex
```

### Output dirs (relative to config file location)

```yaml
output:
  data_dir: data
  report_dir: reports
  log_dir: logs
```

Keep them under `workspace/paper-daily/` by keeping the runtime config there.

### Candidate pool size

```yaml
retrieval:
  candidate_limit: 100
```

Top N for the written report is an **agent** parameter (default 10)—store the user's preference in `research_profile.md` if they change it repeatedly, e.g. `top_n: 5`.

### Proxy

Scripts ignore env proxies by default. To honor `HTTP_PROXY`/`HTTPS_PROXY`:

```powershell
$env:PAPER_DAILY_USE_ENV_PROXY = "1"
```

## Sync with template

If the user wants the template updated too (so new installs get the profile), copy the runtime file over `vendor/paper-daily/config.yaml` only after they confirm.
