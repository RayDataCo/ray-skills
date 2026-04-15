# morning-prep

Calendar-aware daily briefing. Reads your calendar and task board, surfaces prep materials for upcoming meetings, and flags pending homework. Delivers the brief via your configured channel (iMessage, Discord, Slack, or stdout).

## Installation

```bash
mkdir -p ~/.claude/skills/morning-prep
cp SKILL.md ~/.claude/skills/morning-prep/SKILL.md
```

## Required MCP servers

- **Google Calendar MCP** — `list_events` for schedule retrieval
- **One task-board backend** (pick one):
  - Notion MCP
  - Linear MCP
  - `gh` CLI for GitHub Projects
  - Plain markdown TODO (no MCP needed)
- **One delivery-channel MCP** (pick one):
  - iMessage MCP (macOS only)
  - Discord MCP
  - Slack MCP
  - `stdout` (no MCP — prints to session)

## Configuration

Set these environment variables (or inline them in a wrapper script):

```bash
export VAULT_ROOT="$HOME/vault"
export TIMEZONE="America/New_York"               # IANA TZ name
export CALENDAR_IDS="primary"                    # comma-separated if multiple
export TASK_BOARD="notion"                       # notion | linear | github-projects | markdown
export TASK_BOARD_ID="<database-or-project-id>"  # omit for markdown
export DELIVERY_CHANNEL="imessage"               # imessage | discord | slack | stdout
export DELIVERY_TARGET="any;-;+15551234567"      # chat_id / channel / handle
export HOMEWORK_FILE="$VAULT_ROOT/homework.md"
```

### Task-board mapping

The skill expects task records with `title`, `status`, `priority`, `due_date`, `owner`. Map these to whichever backend you use:

- **Notion**: use a database with those properties
- **Linear**: title = Title, priority = Priority, due_date = Due date, status = Status
- **GitHub Projects**: create fields of matching names; query with `gh project item-list`
- **Markdown**: one task per line as `- [ ] {title}  !{priority}  @{owner}  due:{YYYY-MM-DD}`

## Usage

```
/morning-prep                    # today's brief
/morning-prep --date tomorrow    # tomorrow's brief
/morning-prep --date 2026-04-20  # specific date
```

## Scheduling examples

### Claude Code `/loop`

The skill is a one-shot. Schedule it via cron or LaunchAgent rather than `/loop`.

### Cron (weekday 6:30 AM)

```cron
30 6 * * 1-5 /usr/local/bin/claude-code "/morning-prep" >> ~/logs/morning-prep.log 2>&1
```

### LaunchAgent (macOS)

Create `~/Library/LaunchAgents/co.example.morning-prep.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>co.example.morning-prep</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/claude-code</string>
    <string>/morning-prep</string>
  </array>
  <key>StartCalendarInterval</key>
  <array>
    <dict><key>Weekday</key><integer>1</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
    <dict><key>Weekday</key><integer>2</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
    <dict><key>Weekday</key><integer>3</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
    <dict><key>Weekday</key><integer>4</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
    <dict><key>Weekday</key><integer>5</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
  </array>
  <key>StandardOutPath</key><string>/tmp/morning-prep.log</string>
  <key>StandardErrorPath</key><string>/tmp/morning-prep.err</string>
</dict>
</plist>
```

Load with `launchctl load ~/Library/LaunchAgents/co.example.morning-prep.plist`.

### systemd timer (Linux)

```ini
# ~/.config/systemd/user/morning-prep.service
[Unit]
Description=Morning prep brief

[Service]
Type=oneshot
ExecStart=/usr/local/bin/claude-code "/morning-prep"

# ~/.config/systemd/user/morning-prep.timer
[Unit]
Description=Morning prep brief — weekdays 6:30

[Timer]
OnCalendar=Mon..Fri 06:30
Persistent=true

[Install]
WantedBy=timers.target
```

Enable with `systemctl --user enable --now morning-prep.timer`.

## Output

A plain-text brief with sections for calendar, top board priorities, due-date items, pending homework, prep materials (links into the vault), and a suggested-focus line. Delivered via `$DELIVERY_CHANNEL`.

## Timezone note

Google Calendar events come back with both a UTC offset in `dateTime` and a separate `timeZone` label. The offset is the canonical source. Don't double-convert by treating the label as the display timezone.
