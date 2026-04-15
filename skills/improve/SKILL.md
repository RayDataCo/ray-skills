---
description: Read feedback on a skill or process, diarize the mediocre responses, and propose concrete skill rewrites — the self-improvement loop from Garry Tan's "Thin Harness, Fat Skills"
---

# Improve

Read feedback on a skill's performance, identify what was "OK but not good" (not the failures — the mediocre outcomes), and propose concrete rewrites to the skill file. This is Garry Tan's learning loop: survey → investigate → diarize → rewrite the skill.

## When to invoke

- After a batch of work completes using a skill
- When a skill fails or produces mediocre results repeatedly
- When the user says "this isn't working well enough" or "can we improve X"
- On a monthly cadence per active skill

## Process

### 1. Identify the skill to improve

Invoke as: `/improve skill:<skill-name>` or `/improve last-batch` (auto-detects the most recently used skill).

Read the skill file at `~/.claude/skills/<skill-name>/SKILL.md` (or wherever your skills live).

### 2. Gather feedback signals

Look for feedback from multiple sources:

**A. Explicit feedback** — user corrections, "don't do X" messages, approval signals ("perfect, keep doing that"). Check:
- Any auto-memory or feedback log your agent keeps
- Recent conversation history for corrections related to this skill
- Task/ticket notes for comments or blockers encountered during execution

**B. Implicit feedback from results** — check the actual outputs the skill produced:
- What was the error/skip rate?
- Were any outputs misclassified?
- Did the skill catch the edge cases it claims to catch?
- Were any produced artifacts too long/short/off-topic?

**C. "OK but not good" pattern** — Tan's key insight: don't analyze the failures (those are obvious). Analyze the mediocre outcomes — the 3-star reviews, the "fine but not helpful" results. What almost worked but didn't?

### 3. Diarize the feedback

Write a structured profile of what's working and what isn't:

```
SKILL: <name>
RUNS SINCE LAST IMPROVE: <count>
WHAT WORKS WELL:
  - <pattern that consistently produces good results>
  - <pattern the user approved or didn't correct>
WHAT'S MEDIOCRE:
  - <pattern that produces "OK" results — technically correct but not useful>
  - <pattern that requires manual intervention to fix>
WHAT FAILS:
  - <pattern that produces wrong results or errors>
GAP: <what the skill claims to do vs what it actually does>
```

### 4. Propose skill rewrites

For each mediocre or failing pattern, propose a specific change to the skill file:

- **Add a rule** — "When X happens, do Y instead of Z"
- **Refine a threshold** — "Change the skip criteria from 'sales CTA' to 'sales CTA without any extractable technique'"
- **Add a new step** — "After step 3, verify that the output has a mapping section"
- **Remove a step** — "Step 5 adds 10 minutes per run and hasn't been useful — make it opt-in"
- **Change the prompt** — "The sub-agent prompt should include the source-specific gotchas from the discovery note"

Write each proposal as a concrete diff showing old text → new text in the skill file.

### 5. Apply or queue

**If the change is low-risk** (tightening a threshold, adding a step, improving a prompt): apply it directly to the skill file using the Edit tool. Note what changed and why.

**If the change is structural** (removing a mode, changing the architecture, redefining what the skill does): write the proposal to a review doc and flag it for user review. Don't apply structural changes without approval.

### 6. Write the improvement back

After applying changes, add a changelog entry at the bottom of the skill file:

```markdown
## Changelog
- YYYY-MM-DD: <what changed and why, from /improve>
```

This creates an audit trail so future `/improve` runs can see what was already tried.

## The "no one-off work" test

After every `/improve` run, ask: **"If I had to do this improvement again for a different skill, would I do it differently?"** If yes, improve the `/improve` skill itself. The meta-loop.

## Example invocations

- `/improve skill:process-newsletter` — after completing a backfill batch, analyze skip rates, classification accuracy, and output quality across all sources
- `/improve skill:check-board` — after diagnosing a delegation gap, capture the pattern so future skill issues get diagnosed the same way
- `/improve last-batch` — auto-detect the most recent batch and analyze its outputs

## Attribution

This skill is a direct implementation of Garry Tan's "thin harness, fat skills" framing: the harness stays small and deterministic while the skills get fatter over time via a feedback loop. `/improve` is the loop.
