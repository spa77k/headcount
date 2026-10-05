---
name: headcount
description: "Use when deploying to or restarting a service that people stay connected to — game servers (Minecraft, Terraria, Rust, ARK…), voice / TTS / music Discord bots, WebSocket or chat apps, self-hosted web apps. Counts who is actually connected right now, then deploys only when it is empty, at a set time, or after announcing and draining, and verifies persistent data survived. Always use it for requests like \"deploy if nobody is on\", \"when no one is online\", \"apply it at 5pm\", \"without stopping if possible\", \"restart is OK\", 「誰もいなければ反映」「人がいない時に」「pm5:00に反映」「停止せずに反映できるなら」「再起動OK」. Not for services nobody stays connected to (static sites, Cloudflare Pages, serverless functions)."
argument-hint: "[what to deploy and the condition, e.g. \"if nobody is on\" / \"at 22:20\" / \"restart OK\"]"
---

# headcount — count heads before you deploy

Target: $ARGUMENTS

Deploying a service people stay connected to (a game, a voice channel) drops whoever is playing or talking, and can lose anything not yet saved.
This skill counts **who is connected right now** before deploying, and reports — with evidence — how many people were on when the change went live.

## Hard rules

- **If you cannot count, do not assume it is empty.** If no counting method works, say so and stop.
- Stop a service with people on it only when the user said so ("restart is OK", "you can stop it"). Even then, announce first.
- Deploy and restart **the way the project says** (AGENTS.md, CLAUDE.md, README, scripts). If a restart script exists, use it instead of hand-assembling `docker compose up` and the like.
- Never run anything that deletes persistent data (worlds, databases, volumes): no `down -v`, no volume removal, no overwriting data directories.
- When waiting a long time, do not block the conversation with a foreground sleep. Run the wait in the background (a background shell job, a monitor, or the agent's scheduler).

## Steps

### 1. Check whether it can be applied without stopping
If the change can go live through a config reload or hot reload (a plugin/script reload command, re-registering bot commands, `nginx -s reload`, a plain `git pull`), apply it that way. No headcount needed.
If it cannot, state why in one line.

### 2. Pick a counting method and run it once now
Choose from [references/presence.md](references/presence.md). **Run it once and confirm it returns a number** — distinguish "0 people" from "the query failed".
If one service has several places people can be (e.g. in-game players and the bot's voice channel), count all of them.

### 3. Decide when to deploy
Follow the user's wording.

| Wording | Action |
|---|---|
| "if nobody is on" / 「誰もいなければ」 | Count once. 0 → deploy. Otherwise report how many and stop |
| "when nobody is on" / 「誰もいなくなったら」 | Count repeatedly in the background (every 1–5 min); deploy at 0. Set a deadline; if people are still on by then, ask |
| "at 17:00" / 「pm5:00に」 | Wait until then. Count shortly before; if people are on, go to step 4 |
| "restart is OK" / 「再起動OK」 | Count; if people are on, go to step 4 |

- Use the user's timezone. Check the current time with `date` before waiting.
- If the wait may outlive the session, or the machine may sleep, propose a one-shot timer on the server (`systemd-run --on-calendar`, `at`). Do not create a permanent cron job on your own.
- Finish preparing the commit / image / config before waiting, so nothing is left to do afterwards.

### 4. Stopping with people on (only with permission)
1. Announce inside the service (game chat, a bot message), e.g. at 5 min, 1 min and 30 s: when, for how long, and what players should do — in one line.
2. Block new entries (whitelist, maintenance mode, closing sign-ups).
3. Remove whoever is left politely (put the expected return time in the kick reason).
4. **Save while it is still running** (Minecraft: `save-all flush`; databases: a checkpoint). You cannot save after it has stopped.
5. Deploy.

### 5. Safety net before deploying
- Find where persistent data lives (volume, bind mount, DB file) and confirm it survives recreating the container.
- Check the time of the latest backup. If it is stale, ask whether to take one (take it if the project has a defined way).

### 6. Deploy and verify
- Deploy the project's way.
- Verify:
  - It started (healthy, a "started" line in the logs)
  - The intended version is running (production HEAD equals `origin/main`, image digest, reported version)
  - Persistent data is intact (world / DB count or size roughly matches before)
  - The counting method returns a number again
  - Entry restrictions and maintenance mode are lifted
- Follow the project's rules for announcing the deploy. If there are none, announce "we're back" only if people were on.

### 7. Report
```
Deployed <what> at <time>. <N> people were connected at that moment (checked with <method>).
How to check: <one thing the user can do to confirm>
```
If you stopped it, also give the announcement times and how many people were removed. If you did not deploy, give the headcount and what happens next.
