# Ray Skills

A public collection of Claude Code skills from the Ray Data Co operational stack.

## Who is Ray?

Ray is the persona name of the always-on AI COO agent behind Ray Data Co. Ray is an instance of Anthropic's Claude running in a persistent Claude Code environment on a Mac Mini, with a set of skills, MCP servers, and scheduled loops that let it operate autonomously across channels (iMessage, Discord, Gmail, Notion, Google Calendar) while the founder sleeps.

The architecture is the founder's. Ray writes the code.

## How we built these

Each skill in this repo started as an ad-hoc task — "process this newsletter," "brief me on today's calendar" — that we executed once. Then the second time it came up, we asked: if this is going to happen again, where does it live? The result is a discipline Garry Tan calls *"fat skills, thin harness"* — the judgment sits in the skill files, the infrastructure is just plumbing.

These skills are the deterministic, transferable subset of Ray's stack. The rest of Ray's architecture (state management, cross-session memory, the domain-specific playbooks) is proprietary and not shipped here. If you want to see how a one-person consulting practice uses Claude Code as operational backbone, these skills are a real sample of the method.

## Scheduling

Each skill's README has its own cron examples. General pattern: start with `/loop 1h /skill-name` during business hours, tune the interval based on actual signal rate, add a LaunchAgent (or systemd unit on Linux) for persistence across restarts.

## Skills in this repo

Two operational skills (process-newsletter, morning-prep) and two meta-skills (improve, self-review) that compound the usefulness of the agent over time.

- **[process-newsletter](./skills/process-newsletter/)** — ingest email newsletters from whitelisted senders, extract/assess/file with bias and sponsor flagging
- **[morning-prep](./skills/morning-prep/)** — calendar-aware daily briefing; surfaces prep materials for upcoming meetings and pending homework items
- **[improve](./skills/improve/)** — self-rewriting skill flywheel based on Garry Tan's *"thin harness, fat skills"* — reads feedback, diarizes mediocre outcomes, proposes concrete rewrites
- **[self-review](./skills/self-review/)** — rubric-scored quality gate on the agent's own outputs; surfaces drift before it compounds

More skills may be added as they prove stable and transferable.

## See also

- [bwilson668/snowflake-intelligence-demo](https://github.com/bwilson668/snowflake-intelligence-demo) — working Snowflake Cortex + Cortex Analyst + semantic views demo, orchestrated through Claude Code skills, applied to SEC EDGAR data ingestion. A concrete example of the skill-first methodology applied to a specific domain stack.

## License

MIT — see [LICENSE](./LICENSE).

## Contributing

Early-stage repo; no formal contribution guide yet. PRs welcome, open an issue first to discuss. Skills should be generic, runnable without Ray Data Co-specific infrastructure, and include a README with a usage example and a scheduling example.
