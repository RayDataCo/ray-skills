---
description: Process email newsletters from whitelisted senders via Gmail MCP — extract, assess, and file to a local vault with bias/sponsor flagging
---

# Process Newsletter

Ingest email newsletters from whitelisted senders and file them to your vault as structured assessment notes with sponsor/bias disclosure.

## Configuration

Two environment variables drive this skill:

- `VAULT_ROOT` — absolute path to the markdown vault root. Default: `$HOME/vault`.
- `NEWSLETTER_WHITELIST` — path to the whitelist config file. Default: `$VAULT_ROOT/config/newsletter-whitelist.md`.

Output files land in `$VAULT_ROOT/reference/` by default.

### Whitelist format

The whitelist is a plain markdown file with three sections:

```markdown
## Full-ingest senders
- <your-sender-1>@example.com  — format: thought-leadership
- <your-sender-2>@example.com  — format: curation

## Follow-forward-only senders
- <your-sender-3>@example.com  — format: hybrid

## Known sponsor / bias relationships
- <your-sender-1> advises <company-x>  — disclose when <company-x> appears
- <your-sender-2> cross-promotes <your-sender-4>  — flag curation links
```

Senders not on the whitelist are not processed.

## Four modes

- **discovery** — scan one sender's inbox, count unprocessed messages, plan batches, and queue `Backfill <sender> [start-end]` items to your task board. Cheap.
- **batch** — process a specific message-ID range for one sender. Sub-agents fan out per-article so each lives in its own context budget. This is the unit of work the autonomous loop picks up from the task board.
- **backfill** (legacy) — process a sender's full history in a single session. Only use for small senders (<10 messages). For larger backfills, prefer discovery + batch.
- **watch** — poll the inbox for new mail since last run; process matching whitelisted senders.

## Task-board abstraction

"Task board" in this skill means whatever tool the user configured to hold queued work. Supported options:

- **Notion** database (via Notion MCP)
- **Linear** project (via Linear MCP)
- **GitHub Projects** (via `gh` CLI)
- **Plain markdown TODO** file at `$VAULT_ROOT/TODO.md`

The skill writes tasks with fields: `title`, `status`, `priority`, `project`, `notes`. Map these to whichever backend is configured.

## Prerequisite — read the whitelist

Always read `$NEWSLETTER_WHITELIST` before any mode. It is the source of truth for allowed senders, sender format patterns, and known sponsor relationships.

## Mode 1 — Discovery

Invoke as: `Skill: process-newsletter discovery from:<sender-email>`

1. **Search Gmail** via `gmail_search_messages` with `q: "from:<sender>"`. Paginate through all messages.
2. **Skip already-filed messages** — check `$VAULT_ROOT/reference/` for existing files matching `<date>-<sender-slug>-*.md`. Build an "unprocessed message IDs" list sorted oldest-first (so series references resolve in order).
3. **Group into batches of 20 messages.** Last batch can be smaller.
4. **Create one task per batch** on the configured task board. Each task:
   - `title` = e.g. `"Backfill <sender-slug> [1-20]"`
   - `status` = `"To Do"`, `project` = `"Newsletter"`
   - `priority` = inherited from the discovery task's priority
   - `notes` = the message IDs and the `batch:<sender>:<start>-<end>` invocation
5. **File a discovery note** at `$VAULT_ROOT/reference/<YYYY-MM-DD>-backfill-discovery-<sender-slug>.md` with total count, sender-specific gotchas, batch ranges, and task URLs.
6. **Report back** — N batch tasks created, total messages pending, estimated total work.

## Mode 2 — Batch (unit of work)

Invoke as: `Skill: process-newsletter batch:<sender-slug>:<start>-<end>` or read the invocation from a task-board item's notes field.

1. **Fetch the message ID list** from the triggering task's notes field OR re-derive it by querying Gmail for the sender and slicing positions [start..end] oldest-first.
2. **Validate skip list** — re-check `$VAULT_ROOT/reference/` for already-filed articles; remove them from the batch. Idempotent reruns.
3. **Spawn sub-agents in parallel (max 3 concurrent)** using the Agent tool with `subagent_type: general-purpose`. Each sub-agent gets:
   - The sender slug and a single message ID
   - Instructions to follow the **"Process one message"** steps 1-6 below for exactly that ID
   - The target path convention `$VAULT_ROOT/reference/<YYYY-MM-DD>-<sender-slug>-<topic-slug>.md`
   - A concise summary-line return format
4. **Parent session orchestrates** — wait for all sub-agents, collect summary lines, note any skill-shape lessons that emerge (new sponsor pattern, new format variant, new tracked-author candidate).
5. **Update the task board** — mark the batch task `Done`, append a notes summary with files created and any lessons.
6. **If lessons emerged**, update `$NEWSLETTER_WHITELIST` with the new gotcha so future sessions benefit.
7. **If all batches for this sender are complete**, file an auto-generated wrap-up note at `$VAULT_ROOT/reference/<YYYY-MM-DD>-backfill-wrapup-<sender-slug>.md`.

**Why sub-agents:** each newsletter body is ~5-10k tokens of input + ~1k of output. Processing 20 articles sequentially in one parent session burns ~120-220k tokens and hits context pressure. Sub-agents isolate per-article context so the parent stays lean. Empirically ~8 articles per parent session is about the limit without sub-agent delegation.

## Mode 3 — Backfill (legacy; small senders only)

Invoke as: `Skill: process-newsletter backfill from:<sender-email> [limit:<N>]`

Use only when total unprocessed count is <10 messages. Processes sequentially in the current session without spawning sub-agents. For anything larger, use discovery + batch.

## Mode 4 — Watch (ongoing)

Invoke as: `Skill: process-newsletter watch [since:<ISO-date>]`

1. **For each sender in the whitelist:** `from:<sender> after:<since>`.
2. Skip files already present; run "Process one message" on each new result.
3. Append a line to `$VAULT_ROOT/reference/watch-log.md` with what landed.

## Process one message

### Step 1 — Fetch
`gmail_read_message messageId:<id>`. You get headers + plain-text body + any HTML.

### Step 2 — Classify the newsletter format
- **thought-leadership** — one long argument or essay. Single assessment note.
- **curation** — mostly a list of external links with blurbs. Note is "issue summary + deep-dives for items crossing the relevance threshold."
- **hybrid** — argument body + links section at bottom. Both halves in the same note.
- **guest-post** — a guest author hosted by the newsletter owner. Author byline reflects the guest.

### Step 2.5 — Content triage (skip sales-funnel emails)

Skip signals:
- **Pure sales CTA** — primarily selling a course, bootcamp, membership with no extractable technique.
- **"Writing-tagged but actually sales"** — subject has a technique keyword but body is a pitch. Read the first 200 words to verify.
- **Onboarding/drip** — "Welcome," "Day 1 of N," verification codes, receipts.
- **Event/promo** — webinar invites, workshop launches. Skip unless substantive content beyond the CTA.

When in doubt, process. False positives (filing a thin article) are cheaper than false negatives.

### Step 3 — Detect sponsors, self-promo, and bias

Scan for:
- **Explicit sponsor block** — "brought to you by," "sponsored by." Capture the sponsor entity, the relationship type (third-party paid, author-adviser, self-consulting, cross-newsletter promo), and whether disclosure is explicit.
- **Self-consulting CTA** — author pitching their own services. Flag it.
- **Curation-section self-promo** — for EVERY link in a curation section, compare linked domain and author to the sender's. If they match, label self-cross-promo, not real curation.
- **Sister-publication cross-promo** — the "curation" slot given to another newsletter with a relationship. Flag as potentially non-neutral.
- **Affiliate or tracked links** — `utm_source=newsletter`, Substack redirect wrappers, explicit affiliate disclosures.

None of these are automatically disqualifying. The rule is **always disclose them in the note frontmatter and body**.

### Step 4 — Follow links (cautiously, curation mode only)

Deep-fetch a linked article only if ALL of:
1. Link is to a third-party domain (not the same author's blog).
2. Topic clearly crosses a relevance threshold for the user's interests.
3. Title or summary gives a specific, non-generic hook.

Cap at **2 deep-fetches per issue** unless the user expands the budget. Deep-fetch means `WebFetch` with a narrow extraction query. Do not follow paywalled links without explicit access.

### Step 5 — Write the note

File at `$VAULT_ROOT/reference/<YYYY-MM-DD>-<sender-slug>-<topic-slug>.md`.

Frontmatter template:
```yaml
---
date: YYYY-MM-DD
type: reference
source: <newsletter name>
source_url: <canonical post URL if available>
author: <real author, mark guest posts>
newsletter_format: thought-leadership | curation | hybrid | guest-post
sponsored: true | false
sponsor_entity: <sponsor + relationship, if any>
tags: <2-5 topic tags>
---
```

Body sections (adapt per format):
- `# "<Title>" — @<author>` headline
- `## Why this is in the vault` — one sentence on what's being kept and why
- `## ⚠️ Sponsorship` — only if sponsored=true; disclose entity, relationship, and bias implications
- `## The core argument` (thought-leadership) or `## Issue contents` (curation)
- `## Mapping against my work` — the load-bearing section. Where does this reinforce existing discipline, surface a gap, or contradict something already believed? No mapping = no reason to file.
- `## Curation section — notes` — per-link notes, labeling third-party vs self-cross-promo
- `## Related` — wikilinks to connected vault entries

### Step 6 — Cross-link

Add wikilinks in Related pointing to connected prior vault entries. If the newsletter introduces an author worth tracking, add them to a `tracked-authors-candidates.md` list rather than a CRM note directly.

## Copyright discipline

Newsletters are copyrighted. Never paste raw article text into vault notes. Paraphrase, summarize, and quote sparingly — **≤15 words per quote, in quotation marks, with attribution**. Assessment notes should be yours, not the author's words.

## Report back

After batch or watch completes, report:
- Files created (list with paths)
- Senders covered + count per sender
- Any sponsor relationships newly discovered
- Any tracked-author candidates surfaced
- Any link deep-fetches performed (and why)
- Next suggested action

Keep the report under 20 lines unless something surprising happened.
