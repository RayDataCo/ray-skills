# improve

A self-rewriting meta-skill. Reads feedback on another skill's performance, diarizes the mediocre outcomes, and proposes (or applies) concrete rewrites to that skill's SKILL.md. Based on Garry Tan's *"thin harness, fat skills"* essay — the harness stays deterministic, the skills compound.

## What it does

`/improve` runs a four-step flywheel on a named skill:

1. **Survey** — gather feedback signals (explicit corrections, implicit output quality, silent approvals).
2. **Investigate** — look past the obvious failures and focus on the "OK but not good" middle — Tan's key insight.
3. **Diarize** — write a structured profile of what works, what's mediocre, what fails, and the gap between claim and reality.
4. **Rewrite** — propose concrete diffs to the skill file; apply low-risk changes directly, queue structural ones for review.

## When to invoke

- After a batch of work completes using a skill (e.g. a 400-email newsletter backfill)
- When a skill produces mediocre results repeatedly
- When the user explicitly says "this isn't working well enough"
- On a monthly cadence per active skill (recommended)

## Required inputs

The skill pulls feedback from whatever surfaces the agent has available:

- Conversation history with the user (corrections, approvals)
- An auto-memory / feedback log if one exists
- The actual outputs the skill produced (files, notes, tasks)
- Any ticket/task system the agent writes to

If none of those exist, the skill will prompt for feedback rather than hallucinate signals.

## Example: improving a hypothetical `summarize-podcast` skill

```
$ /improve skill:summarize-podcast

SKILL: summarize-podcast
RUNS SINCE LAST IMPROVE: 23
WHAT WORKS WELL:
  - Transcript extraction is clean (99% success)
  - Episode metadata is always correctly parsed
WHAT'S MEDIOCRE:
  - Summaries are technically accurate but lead with the interviewee's bio
    instead of the actual insight — user consistently rewrites the opening
  - Speaker attribution in quotes is inconsistent (sometimes "Host:", sometimes name)
WHAT FAILS:
  - Episodes over 2hrs get truncated silently
GAP: skill claims to produce "assessment notes" but the outputs read like
     Wikipedia summaries — no assessment, no opinion, no mapping to the user's
     current work.

PROPOSED CHANGES (low-risk, applying directly):
  - Add step: "Lead with the single most surprising claim, not the bio"
  - Add rule: "Always use speaker's first name in attribution"
  - Add threshold: "If transcript > 50k tokens, chunk and summarize per segment"

PROPOSED CHANGES (structural, queued for review):
  - Redefine output contract: summaries → assessment notes with a required
    "Why this is worth your time" section
```

Changelog entry is appended to the skill, and a review doc is written for the structural change.

## Scheduling

Monthly per skill is a reasonable default — enough runs accumulate to see patterns, not so often that noise dominates signal.

```cron
# Run /improve on the 1st of each month at 2 AM, rotating through active skills
0 2 1 * * claude -p "/improve rotate-active-skills"
```

Or couple it with a batch: run `/improve skill:X` automatically whenever skill X finishes a batch of ≥ 20 runs.

## Why this exists

Most AI agents degrade silently — the same mistake recurs because nothing closes the loop between "user was mildly annoyed" and "skill file got edited." `/improve` is the explicit loop. The "no one-off work" test at the end of each run (would I do this improvement differently next time? if yes, improve `/improve` itself) means the meta-skill also compounds.
