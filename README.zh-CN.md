# headcount

[English](README.md) | [日本語](README.ja.md) | **简体中文** | [한국어](README.ko.md)

一个在部署之前先数一数当前有多少人在线的 Agent Skill。

在有人使用时部署游戏服务器或语音机器人，会让正在游戏或通话的人直接掉线，还可能丢失尚未保存的数据。装上 `headcount` 后，你的编码 Agent 会先确认真实的在线人数，然后只在服务为空、到达指定时间、或已公告并让玩家退出之后才部署。部署完成后，它还会确认持久化数据是否完好。

支持 Claude Code、Codex、Antigravity（`agy`），以及所有支持 [Agent Skills](https://agentskills.io) `SKILL.md` 格式的 Agent。

## 为什么要用

- **每次部署前不用再问“现在有人吗？”** 由 Agent 直接向服务器发起真实查询，不必你自己去翻玩家列表或语音频道。
- **不用干等。** 说一句“等没人了再部署”，然后去做别的事。它会在后台持续计数，服务一空就立刻部署。
- **定好时间，安心去睡。** “22:20 部署，可以重启”：它会等到点，临近时先数人；如果还有人，就先公告并让他们退出，再部署。
- **检查失败不会被当成“没人”。** 数不出来就停下，绝不当作 0。
- **留有凭证。** 报告会写明部署那一刻有多少人在线、是怎么数的，以及数据是否完好。

## 适用场景

- **Minecraft 服务器。** 更新插件或配置时不会把正在建造的玩家踢下线，重启前会先保存。
- **Discord 语音 / TTS / 音乐机器人。** 发布新版本时不会打断正在进行的通话。
- **Terraria、Rust、ARK、CS2 等游戏服务器。** 补丁重启选在服务器空闲时，而不是在团战中途。
- **WebSocket / 聊天 / 实时应用。** 在没有打开的连接时发布。
- **自托管的 Web 应用。** 根据访问日志或会话表，挑一个空闲时段。

不适用于没有人保持连接的服务（静态网站、无服务器函数），因为没有可数的对象。

## 它做什么

1. 如果重新加载即可生效，就不重启直接应用变更。
2. 选择一种统计在线人数的方法并实际运行一次，确认可用。查询失败绝不会被当作“0 人”。
3. 根据你的说法决定如何部署：
   - “如果没人在线”：数一次，只有 0 人时才部署
   - “等没人了”：在后台持续计数，0 人时部署（有截止时间）
   - “22:20”：等到那个时间，临近时再数
   - “可以重启”：公告 → 禁止新玩家进入 → 让玩家退出 → 在运行中保存 → 部署
4. 确认服务正常运行、版本正确、数据仍在、进入限制已解除。
5. 报告部署时有多少人在线，以及统计方法。

## 支持的统计方式

| 服务 | 方法 |
|---|---|
| Minecraft（Java 版、经 Geyser 的基岩版） | RCON `list` |
| Terraria | 控制台 `playing`、TShock REST `/v2/players/list` |
| Rust、ARK、CS2 等 Steam 游戏 | A2S 查询、Rust WebRCON |
| Discord 语音 / TTS / 音乐机器人 | `GET /guilds/{id}/voice-states/@me` 以及机器人自身状态 |
| WebSocket / SSE / TCP 应用 | `ss` 统计已建立的连接 |
| Web 应用 | 近期访问日志中的 IP、会话表 |

详情见 [skills/headcount/references/presence.md](skills/headcount/references/presence.md)

## 安装

### 一次装给所有 Agent

```bash
npx skills add spa77k/headcount -g
```

使用 [skills CLI](https://github.com/vercel-labs/skills)，会询问要安装到哪些 Agent。

### 手动安装

```bash
git clone https://github.com/spa77k/headcount.git
```

然后把 `skills/headcount` 复制或软链接到你的 Agent 的 skill 目录：

| Agent | 全局 | 按项目 |
|---|---|---|
| Claude Code | `~/.claude/skills/headcount` | `.claude/skills/headcount` |
| Codex | `~/.agents/skills/headcount` | `.agents/skills/headcount` |
| Antigravity CLI（`agy`） | `~/.gemini/antigravity-cli/skills/headcount` | `.agents/skills/headcount` |
| Antigravity IDE | `~/.gemini/config/skills/headcount` | `.agents/skills/headcount` |

## 使用方法

像下面这样提出请求即可触发：

```
如果服务器没人，就部署这个。
22:20 应用，可以重启。
等语音频道没人了，再推送机器人更新。
```

也可以直接调用：`/headcount 如果没人在线就部署`。

Skill 不会自己编造部署流程。请把你的流程（SSH 主机、重启脚本、容器名等）写进 `AGENTS.md` 或 `CLAUDE.md`，Skill 会照着执行。

## 许可证

MIT
