---
description: Calendar-aware daily briefing — surfaces prep materials for upcoming meetings, pending homework, and the top priorities on the task board
---

# Morning Prep

Read the user's calendar and task board to generate a daily action brief. Proactively prep materials for upcoming meetings and surface what needs attention today.

## Configuration

- `VAULT_ROOT` — absolute path to the markdown vault. Default: `$HOME/vault`.
- `TIMEZONE` — IANA timezone string. Default: `America/New_York`.
- `CALENDAR_IDS` — comma-separated calendar IDs to query. Default: `primary`.
- `TASK_BOARD` — one of `notion`, `linear`, `github-projects`, `markdown`. Default: `markdown` (reads `$VAULT_ROOT/TODO.md`).
- `TASK_BOARD_ID` — database/project ID for the configured backend (not needed for `markdown`).
- `DELIVERY_CHANNEL` — one of `imessage`, `discord`, `slack`, `stdout`. Default: `stdout`.
- `DELIVERY_TARGET` — chat_id / channel / handle for the delivery channel.
- `HOMEWORK_FILE` — path to a markdown file listing pending homework items. Default: `$VAULT_ROOT/homework.md`.

## Usage

`/morning-prep [--date <YYYY-MM-DD>]`

Default: today. `--date tomorrow` looks ahead.

## Process

### 1. Check the clock

Run `date` to get the current date and time. Never infer.

### 2. Read the calendar

Query `list_events` from the Google Calendar MCP for today and tomorrow:
- `timeMin`: start of target date
- `timeMax`: end of next day
- `timeZone`: `$TIMEZONE`

For each event extract: title, time, duration, attendees, description/notes, whether it's a recurring standup vs a one-off.

**Timezone note:** Google Calendar returns events with a `dateTime` containing a UTC offset AND a separate `timeZone` label. The offset is canonical. The `timeZone` label is organizer metadata, not the display timezone. Convert from the offset directly to `$TIMEZONE`.

### 3. Classify events by prep needs

| Event type | Prep action |
|------------|-------------|
| **Decision meeting** (keywords: decision, review, eval, negotiate) | Find the relevant decision doc in the vault. Surface open questions. Remind the user of homework items. |
| **Client/external call** (attendees on outside domains) | Check vault for prior notes on this person/company. Surface relevant context. |
| **Content deadline** (keywords: publish, draft, deadline, due) | Check if research brief / draft is ready. Flag if not. |
| **Recurring standup** (daily/weekly pattern) | No prep — just note it in the schedule. |
| **Interview / hiring** (keywords: interview, candidate) | Surface prior notes on candidate or role. |
| **Unknown** | List it, no prep action. |

### 4. Check the task board

Query the configured task board for:
- Tasks due today or tomorrow
- Tasks in "Blocked" status that might be unblocked by today's meetings
- The top 3 priority tasks owned by the user

### 5. Check pending homework

Search `$HOMEWORK_FILE` and recent vault entries for items explicitly flagged as pending homework:
- Anything under a `## Pending` or `## Homework` heading
- Any line tagged `#homework` or `#todo-owner`
- Anything in the vault modified in the last 7 days containing "needs to" / "still need to" near the user's name

### 6. Generate the brief

```
Good morning. Here's the brief for {day}, {date}.

CALENDAR
--------
{time} — {event title} ({duration})
  Prep: {action or "none needed"}

BOARD — TOP PRIORITIES
--------
1. {task} — {status} ({priority})
2. {task} — {status} ({priority})
3. {task} — {status} ({priority})

{if due-date tasks exist}
DUE TODAY/TOMORROW
--------
- {task} — due {date}

{if homework items exist}
HOMEWORK (still pending)
--------
- {item}

{if prep materials were generated}
PREP MATERIALS
--------
- {doc title}: {path or summary}

SUGGESTED FOCUS
--------
{1-2 sentences on the highest-leverage use of today based on calendar + board + pending items}
```

### 7. Deliver

Dispatch the brief via `$DELIVERY_CHANNEL`:
- `imessage` — use the iMessage MCP `reply` tool with `chat_id=$DELIVERY_TARGET`
- `discord` — use the Discord MCP `reply` tool with `chat_id=$DELIVERY_TARGET`
- `slack` — use the Slack MCP `slack_send_message` with the configured channel
- `stdout` — print to the session

If prep materials were generated, attach or link them. Optionally post a shorter version to a secondary channel for the record.

## Scheduling

Run as part of a morning autonomous loop, timed before the user's typical wake-up:
- **Weekdays**: 6:30 AM (in `$TIMEZONE`)
- **Weekends**: skip unless calendar has events

Manual trigger: `/morning-prep` anytime.

See `README.md` in this directory for cron / LaunchAgent / systemd examples.

## Integration points

- **Google Calendar MCP**: `list_events` for schedule
- **Task board** (Notion / Linear / GitHub Projects / markdown): top priorities + due dates
- **Vault search**: for meeting-attendee context and decision docs
- **Delivery channel MCP**: iMessage / Discord / Slack

## Example output

```
Good morning. Here's the brief for Tuesday, April 14.

CALENDAR
--------
10:00 AM — Call with <contact> (30 min)
  Prep: Decision doc at projects/<topic>/decision-analysis.md
  9 questions prepped. 5 homework items — see below.

HOMEWORK (still pending)
--------
- Bank statement pull (needed for runway verification)
- Healthcare plan verification
- Employment agreement review
- Tax status confirmation
- Read <article> (context for positioning)

SUGGESTED FOCUS
--------
The 10 AM call is the gating event. Complete the bank pull and
healthcare verification before it so you have the numbers fresh.
The 9 questions in the decision doc are ready — review over coffee.
```
