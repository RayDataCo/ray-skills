# process-newsletter

Ingest email newsletters from whitelisted senders via the Gmail MCP, extract them into structured assessment notes with sponsor and bias disclosure, and file them to a local markdown vault.

## Installation

Copy the skill into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/process-newsletter
cp SKILL.md ~/.claude/skills/process-newsletter/SKILL.md
```

Claude Code will auto-discover it on next session.

## Required MCP servers

- **Gmail MCP** — for `gmail_search_messages` and `gmail_read_message`. The skill assumes the standard Anthropic Gmail MCP tool names.
- **A task-board MCP** (optional but recommended for batch-mode work queuing). Pick one:
  - Notion MCP
  - Linear MCP
  - GitHub CLI (`gh`) for GitHub Projects
  - Plain markdown TODO file (no MCP needed)

## Configuration

### Environment variables

```bash
export VAULT_ROOT="$HOME/vault"                                       # where notes land
export NEWSLETTER_WHITELIST="$VAULT_ROOT/config/newsletter-whitelist.md"
```

### Whitelist file

Create `$NEWSLETTER_WHITELIST` with this structure:

```markdown
## Full-ingest senders
- <your-sender-1>@example.com  — format: thought-leadership
- <your-sender-2>@example.com  — format: curation
- <your-sender-3>@example.com  — format: hybrid

## Follow-forward-only senders
- <your-sender-4>@example.com  — format: thought-leadership

## Known sponsor / bias relationships
- <your-sender-1> advises <company-x>
- <your-sender-2> regularly cross-promotes <your-sender-5>
- <your-sender-3> sells a course on <topic>; treat related mid-article CTAs as self-promo
```

Senders not listed are ignored. The relationships section feeds Step 3 of per-message processing (sponsor detection).

## Usage

Four modes:

```
Skill: process-newsletter discovery from:<sender-email>
Skill: process-newsletter batch:<sender-slug>:<start>-<end>
Skill: process-newsletter backfill from:<sender-email> [limit:<N>]
Skill: process-newsletter watch [since:<ISO-date>]
```

Typical flow for a new sender:

1. Run `discovery` to survey the inbox and queue batch tasks
2. Let the autonomous loop (or you, manually) pick up each batch from the task board
3. Run `watch` on a schedule to catch new mail

## Scheduling examples

### Using Claude Code's `/loop`

```
/loop 1h /process-newsletter watch
```

### Cron (macOS / Linux)

```cron
# Watch every hour during business hours
0 9-18 * * 1-5 /usr/local/bin/claude-code "Skill: process-newsletter watch" >> ~/logs/newsletter-watch.log 2>&1
```

### LaunchAgent (macOS persistent)

Create `~/Library/LaunchAgents/co.example.newsletter-watch.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>co.example.newsletter-watch</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/claude-code</string>
    <string>Skill: process-newsletter watch</string>
  </array>
  <key>StartCalendarInterval</key>
  <dict><key>Minute</key><integer>0</integer></dict>
  <key>StandardOutPath</key><string>/tmp/newsletter-watch.log</string>
</dict>
</plist>
```

Load with `launchctl load ~/Library/LaunchAgents/co.example.newsletter-watch.plist`.

## Output

Each processed message becomes a file at:

```
$VAULT_ROOT/reference/<YYYY-MM-DD>-<sender-slug>-<topic-slug>.md
```

Frontmatter includes `sponsored`, `sponsor_entity`, `newsletter_format`, and topic tags. The body always includes a "Mapping against my work" section — if you can't write that section, the note doesn't get filed.

## Copyright discipline

Newsletters are copyrighted. The skill quotes **≤15 words** at a time with attribution; assessment notes summarize and paraphrase. Do not loosen this constraint.
