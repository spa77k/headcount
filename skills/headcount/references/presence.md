# How to count who is connected

Run each method once and confirm it returns a number before relying on it. Never read a failure (connection refused, empty response) as zero.
If the project already has a command or endpoint that reports the headcount, prefer it.

## Game servers

| Target | Method | What zero looks like |
|---|---|---|
| Minecraft Java (Paper etc.) | RCON `list` (with the itzg Docker image: `docker exec <container> rcon-cli list`) | `There are 0 of a max of N players online` |
| Minecraft Bedrock players via Geyser | Included in the Java-side `list` (names carry the Floodgate prefix) | Same as above |
| Terraria (vanilla) | Server console `playing` (no slash). Under tmux/screen, use `send-keys` and read the output. Join/leave lines in the log also work if your setup logs them | No player names |
| Terraria (TShock) | REST API `/v2/players/list`, or `playing` in the console | Empty list |
| Steam games (Rust, ARK, CS2…) | A2S query (`player_count` from `a2s.info()` in `python-a2s`). The query port may differ from the game port | `player_count` is 0 |
| Rust | WebRCON `playerlist` | Empty array |

Communities that play together often sit in a Discord voice channel too. If yours does, also count with the bot methods below.

## Discord bots

- **Voice / TTS / music bots**: check whether the bot itself is in a voice channel with `GET /guilds/{guild.id}/voice-states/@me` (404 if not). If it is, count the people in that channel from the bot's internal state or logs. Other users' voice state can be read one at a time with `GET /guilds/{guild.id}/voice-states/{user.id}` (the bot needs permission to connect to that channel).
- **Bots with ongoing interactions** (tickets, an AI reply being generated, an open poll): count in-flight work from the bot's DB or logs. Stopping mid-generation loses that reply.
- REST has no endpoint that lists everyone in voice. If the bot has a headcount command or health endpoint, use it.

## Web apps and WebSockets

| Target | Method |
|---|---|
| Long-lived connections (WebSocket, SSE, raw TCP game traffic) | On the server: `ss -Htn state established '( sport = :<port> )' \| wc -l`. Behind a reverse proxy, count on the proxy's port |
| Ordinary web apps | Distinct IPs in the access log over the last 5–10 minutes, excluding health checks, bots and yourself |
| Signed-in users | Last-seen timestamps in the app's session table |

## When nothing can count

- If no method works, do not treat the service as empty. Ask the user whether to deploy without knowing the headcount.
- For a service deployed often, suggest adding a command that reports the headcount (suggest only; do not build it unasked).
