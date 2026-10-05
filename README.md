# headcount

**English** | [日本語](README.ja.md)

An agent skill that counts who is connected before it deploys.

Deploying a game server or a voice bot while people are using it drops them mid-game or mid-call, and can lose unsaved data. `headcount` makes your coding agent check the real headcount first, then deploy only when the service is empty, at a set time, or after announcing and draining. Afterwards it verifies that persistent data survived.

Works with Claude Code, Codex, Antigravity (`agy`) and any agent that supports the [Agent Skills](https://agentskills.io) `SKILL.md` format.

## Why use it

- **No more "is anyone on?" before every deploy.** The agent asks the server itself, with a real query, instead of you opening a player list or a voice channel.
- **Deploy without waiting around.** Say "when nobody is on" and go do something else. It counts in the background and deploys the moment the service is empty.
- **Schedule it and sleep.** "Apply it at 22:20, restart is OK" waits, counts shortly before, and if people are still on, announces and drains them first.
- **A broken check never turns into "empty".** If the count fails, it stops. It does not assume 0.
- **You get proof.** The report says how many people were connected at the moment of deploy and how they were counted, and that your data survived.

## Where it pays off

- **Minecraft server.** Update plugins or config without kicking someone mid-build, with a save before any restart.
- **Discord voice / TTS / music bot.** Ship a new version without cutting off a call.
- **Terraria, Rust, ARK, CS2 and other game servers.** Restart for a patch when the server is empty, not in the middle of a raid.
- **WebSocket / chat / realtime apps.** Roll out when no sockets are open.
- **Self-hosted web apps.** Pick a quiet moment from the access log or session table.

Not for services nobody stays connected to (static sites, serverless functions). There is nothing to count.

## What it does

1. Applies the change without a restart when a reload is enough.
2. Picks a way to count connected people and runs it once to prove it works. A failed query is never read as "0".
3. Deploys based on your wording:
   - "if nobody is on": count once, deploy only at 0
   - "when nobody is on": count in the background and deploy at 0, with a deadline
   - "at 17:00": wait, then count shortly before
   - "restart is OK": announce → block new entries → remove players → save while running → deploy
4. Checks the service is healthy, runs the intended version, still has its data, and has entry restrictions lifted.
5. Reports how many people were connected at the moment of deploy, and how they were counted.

## Supported ways to count

| Service | Method |
|---|---|
| Minecraft (Java / Bedrock via Geyser) | RCON `list` |
| Terraria | Console `playing`, TShock REST `/v2/players/list` |
| Rust, ARK, CS2 and other Steam games | A2S query, Rust WebRCON |
| Discord voice / TTS / music bots | `GET /guilds/{id}/voice-states/@me` and the bot's own state |
| WebSocket / SSE / TCP apps | `ss` established connections |
| Web apps | Recent access log IPs, session table |

Details: [skills/headcount/references/presence.md](skills/headcount/references/presence.md)

## Install

### All agents at once

```bash
npx skills add spa77k/headcount -g
```

Uses the [skills CLI](https://github.com/vercel-labs/skills). It asks which agents to install into.

### Manually

```bash
git clone https://github.com/spa77k/headcount.git
```

Then copy or symlink `skills/headcount` into your agent's skill folder:

| Agent | Global | Per project |
|---|---|---|
| Claude Code | `~/.claude/skills/headcount` | `.claude/skills/headcount` |
| Codex | `~/.agents/skills/headcount` | `.agents/skills/headcount` |
| Antigravity CLI (`agy`) | `~/.gemini/antigravity-cli/skills/headcount` | `.agents/skills/headcount` |
| Antigravity IDE | `~/.gemini/config/skills/headcount` | `.agents/skills/headcount` |

## Usage

The skill triggers on requests like these:

```
Deploy this if nobody is on the server.
Apply it at 22:20. Restart is OK.
Push the bot update once nobody is in voice.
```

Or call it directly: `/headcount deploy if nobody is on`.

The skill does not invent a deploy procedure. Put yours (SSH host, restart script, container names) in `AGENTS.md` or `CLAUDE.md`, and the skill will follow it.

## License

MIT
