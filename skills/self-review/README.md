# self-review

A rubric-scored quality gate on your agent's own outputs. Scans recent artifacts (vault notes, briefs, summaries), scores them against a 13-point rubric, flags the weak ones, and optionally fixes them.

## What it does

Most agents that write to a knowledge base degrade silently: the 50th entry looks worse than the 5th, but no one notices because no one re-reads them. `/self-review` re-reads them. It produces:

- A per-entry score (0-13) and letter grade (A/B/C/D)
- A list of entries that need attention, with the specific failed criteria
- Systemic patterns (e.g. "mapping sections are getting more generic across the last 10 entries")
- Optionally, automated fixes for the mechanical failures (missing frontmatter, missing cross-links)

## The 13-point rubric

| # | Criterion | Weight |
|---|-----------|--------|
| 1 | Frontmatter complete (date, type, source, author, tags) | 2 |
| 2 | Why-in-scope section present and specific | 3 |
| 3 | Mapping against your project's theses/goals, actionable | 3 |
| 4 | Cross-links — at least 2 wikilinks to real entries | 2 |
| 5 | Bias/sponsor flagging when source is sponsored or biased | 1 |
| 6 | No copy-paste walls from source material | 1 |
| 7 | Conciseness — file under 300 lines | 1 |

Max 13 points. Grades: A (12-13), B (10-11), C (8-9), D (<8).

The rubric is the transferable IP — the weights encode that the hardest thing to get right is not *whether* an entry exists but *why it exists and what it connects to*. Frontmatter is 2 points because it's mechanical and fixable. The mapping section is 3 points because that's where judgment lives.

## Configuration

- `VAULT_ROOT` env var — which directory to scan
- `--since`, `--limit` flags — control scope per run
- `--fix` flag — auto-apply mechanical fixes; omit for report-only mode
- Rubric can be edited in `SKILL.md`. Common adjustments:
  - Swap "mapping against project theses" for your domain-specific equivalent (active research questions, client engagements, OKRs, etc.)
  - Drop the sponsor criterion if you don't ingest sponsored sources
  - Add a domain-specific criterion (e.g. "has a falsifiable prediction" for research notes)

## Example output

```
Self-Review Report — 2026-04-08 to 2026-04-15
Entries reviewed: 14

Grade distribution:
  A: 3 (21%)
  B: 6 (43%)
  C: 4 (29%)
  D: 1 (7%)

Average score: 9.8/13 (down from 10.4 last week)

Entries needing attention:
1. 2026-04-12-some-newsletter-issue.md — Score 7/13 (Grade D)
   Issues: missing mapping section, no cross-links, sponsor field blank
2. 2026-04-11-conference-recap.md — Score 8/13 (Grade C)
   Issues: mapping section vague ("relevant to our work"), only 1 cross-link

Top entries (for template reference):
1. 2026-04-14-data-quality-essay.md — 13/13
2. 2026-04-09-benchmark-analysis.md — 12/13

Systemic patterns:
- 3 of 4 newsletter entries this week missed the sponsor field — check the
  producing skill's prompt
- Mapping sections trending more generic — 2 entries used the literal phrase
  "relevant to our work"
```

When run with `--fix`, the mechanical failures (missing frontmatter, missing cross-links) get patched in place, and the report notes what was fixed.

## Scheduling

Weekly is the right cadence for most setups — enough new entries to see a trend, not so many that the review itself becomes a chore.

```cron
# Run /self-review every Monday at 5 AM, report-only
0 5 * * 1 claude -p "/self-review --since 7d"
```

Pair with `/improve`: when self-review surfaces a systemic pattern pointing to a producing skill, queue an `/improve` run on that skill.

## Why this exists

Quantity of output is cheap with an AI agent. Quality decays without an explicit feedback mechanism. `/self-review` is that mechanism — a gate the outputs have to pass periodically, scored against criteria the user actually cares about.
