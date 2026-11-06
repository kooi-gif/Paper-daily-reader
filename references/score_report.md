# Score + day-report module

Load after a successful fetch (or when the user asks to score / write the daily report).

Read `data/processed/YYYY-MM-DD_candidates.json` and optional private notes in `research_profile.md`.

## Scoring rubric (each 0–5)

| Dimension | Meaning |
|---|---|
| Methodological Relevance | Method fit to the research profile |
| Inspiration Value | Novel mechanism / surprising insight |
| Transferability | Can the idea move into our problems |
| Paper Quality | Clarity, experiments, rigor (from available evidence) |
| Novelty / Timeliness | Newness vs concurrent work (abstract/page only if no PDF) |
| Actionability | Concrete next experiment / module |

**Final Score** =

```text
0.25 * Methodological Relevance
+ 0.25 * Inspiration Value
+ 0.20 * Transferability
+ 0.15 * Paper Quality
+ 0.10 * Novelty / Timeliness
+ 0.05 * Actionability
```

Do not boost a paper solely because it contains domain keywords. Adjacent-field papers may score high if methods transfer.

## Procedure

1. Score **every** candidate from title + abstract + venue/categories + `retrieval_reason` (ignore raw keyword_score as authority).
2. Write `data/processed/YYYY-MM-DD_scored.json` (schema below).
3. Pick ~**15** highest for deeper read (`selected_for_deep_read`).
4. For final **Top N** (default 10, user may change): open abs page or PDF when possible; read abstract, intro, method, experiments, conclusion.
5. Set `reading_evidence` per paper: `abstract-only` | `paper page` | `pdf`.
6. Write `reports/YYYY-MM-DD.md`.
7. Reply to the user with: candidate count, Top N titles, scored path, report path, failures / abstract-only limits.

### scored.json shape

```json
{
  "date": "YYYY-MM-DD",
  "status": "ok",
  "generated_at": "...",
  "candidate_path": "...",
  "candidate_count": 0,
  "deep_read_ids": [],
  "top10_ids": [],
  "top_n": 10,
  "papers": [
    {
      "...candidate fields": "...",
      "semantic_scores": {
        "methodological_relevance": 0,
        "inspiration_value": 0,
        "transferability_to_my_research": 0,
        "paper_quality": 0,
        "novelty_timeliness": 0,
        "actionability": 0,
        "final_score": 0
      },
      "selected_for_deep_read": false,
      "selected_for_top10": false,
      "reading_evidence": "abstract-only",
      "semantic_rank": 0
    }
  ]
}
```

## Report template (`reports/YYYY-MM-DD.md`)

```markdown
# 论文日报 YYYY-MM-DD

- 候选数：N
- 证据范围：以各篇 reading_evidence 为准
- 配置：workspace/paper-daily/config.yaml

## 今日概览
2-4 句趋势概括（基于候选池，不编造）。

## Top {N} Papers

### 1. <title> — final score X.XX
- 链接 / 来源 / 分类
- 核心方法
- 迁移价值
- 可转化实验或模块
- 局限
- 阅读建议
- 证据：abstract-only | paper page | pdf（引用页码若可得）

...

## 今日重点趋势
bullets

## 潜在研究 idea
bullets（标注假设性）

## 值得加入 related work
bullets with IDs
```

## Prohibited

- Ending after only running the script
- Keyword-template “recommendations” without reading candidates
- Fabricated metrics, venues, or PDF claims
- Padding with repetitive boilerplate
- Auto git push

## Handoff to deep architecture analysis

After the user picks IDs from the report, switch to **embodied-paper-assistant** (`references/download_read.md` / `analyze.md`) for PDF download, architecture briefs, or deep diagrams—do not re-implement that pipeline here.
