# Self-Review — Output Quality Scoring

Automatically review recent outputs (vault entries, notes, briefs, summaries — whatever artifact your agent produces) for quality, surface the ones that fall short, and fix them proactively. This is the feedback loop that prevents drift — the body of work should get better over time, not just bigger.

## Usage

`/self-review [--since <date>] [--limit <N>] [--fix]`

- `--since`: review entries created after this date (default: 7 days ago)
- `--limit`: max entries to review (default: 20)
- `--fix`: automatically fix issues found (default: report only)

Configure which directory to scan via a `VAULT_ROOT` env var (or adapt the path in step 1 to your own output location).

## Process

### 1. Gather recent entries

Scan `$VAULT_ROOT` (or your configured output root) for files created since the target date. Sort by date, newest first. Skip raw source material (e.g. transcripts, clippings) — review only synthesized outputs.

### 2. Score each entry

Read each entry and score against these criteria:

| Criterion | Weight | Pass condition |
|-----------|--------|----------------|
| **Frontmatter complete** | 2 | Has date, type, source, author, tags. Any type-specific required fields present (e.g. sponsored flag for newsletter entries). |
| **Why-in-scope section** | 3 | Has a "Why this is in the vault" (or equivalent) section that justifies filing. Not generic — specific ("reinforces the data-moat finding from cross-check"). |
| **Mapping section** | 3 | Has a mapping against your project's theses/goals with specific, actionable connections. Fails if missing or filled with vague statements like "this is relevant to our work." |
| **Cross-links present** | 2 | Has a Related section with at least 2 wikilinks to existing entries. Links must resolve to real files. |
| **Bias/sponsor flagging** | 1 | For entries derived from sponsored or biased sources: sponsor field populated, sponsor entity named, bias disclosed in body. |
| **No copy-paste walls** | 1 | No paragraphs that appear to be verbatim copy-paste from source material (>30 consecutive words matching a common article structure). Summaries should be original. |
| **Conciseness** | 1 | File is <300 lines. Outputs should be assessments, not reproductions. |

**Scoring:**
- Each criterion: 0 (fail) or full weight (pass)
- Max score: 13
- Grade: A (12-13), B (10-11), C (8-9), D (<8)

Adjust the rubric for your domain: replace "mapping section" with whatever the equivalent is for your project (e.g. for a research vault, "mapping against active research questions"; for a consulting practice, "mapping against active client engagements").

### 3. Generate report

```
Self-Review Report — {date range}
Entries reviewed: {N}

Grade distribution:
  A: {count} ({%})
  B: {count} ({%})
  C: {count} ({%})
  D: {count} ({%})

Average score: {X}/13 ({trend vs last review if available})

Entries needing attention:
1. {filename} — Score {X}/13 (Grade {G})
   Issues: {list of failed criteria}

2. {filename} — Score {X}/13 (Grade {G})
   Issues: {list of failed criteria}

Top entries (for template reference):
1. {filename} — Score 13/13
2. {filename} — Score 12/13

Systemic patterns:
- {e.g., "5 of 8 entries missing sponsor field" or "mapping sections are getting more generic"}
```

### 4. Fix (if --fix flag)

For each entry scoring C or below:
- **Missing frontmatter fields**: add them (look up source to fill correctly)
- **Missing cross-links**: search the output root for related entries and add wikilinks
- **Weak mapping section**: rewrite with specific connections to your project's theses
- **Missing why-in-scope**: add based on the content and its context

Do NOT fix:
- Copy-paste issues (requires re-reading source and rewriting — flag for manual review)
- Entries where the content itself is thin (the source may just not have been worth filing — flag for possible archiving)

### 5. Track trends

Append results to a review log (e.g. `$VAULT_ROOT/self-review/review-log.md`):

```markdown
## {date}
- Entries reviewed: {N}
- Average score: {X}/13
- Grade distribution: A:{n} B:{n} C:{n} D:{n}
- Fixed: {n} entries
- Systemic issues: {list}
```

This creates a longitudinal record of output quality over time.

## Scheduling

Run weekly, or after any batch ingestion. Can also be triggered manually.

## Integration with /improve

When self-review finds systemic patterns (e.g., "entries consistently have weak mapping sections"), it should:
1. File the pattern as a learning in the review log
2. If the pattern points to a skill deficiency (e.g., the producing skill's mapping prompt is too vague), create a task for `/improve` to address it

This closes the loop: self-review catches drift → improve fixes the skill → future entries score higher.

## Related

- `/improve` — fixes skill deficiencies that self-review surfaces
- `/vault-health` — structural health (orphans, broken links, index drift) — complementary, not a substitute
- `/cross-check` — content-level contradictions and gaps across entries
