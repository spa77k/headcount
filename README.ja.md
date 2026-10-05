# headcount

[English](README.md) | **日本語**

反映する前に、いまつながっている人を数えるエージェント用スキルです。

ゲームサーバーやボイスBotを使っている最中にデプロイすると、遊んでいる人や話している人が切断され、保存前のデータが消えることもあります。`headcount` を入れると、エージェントは先に実際の人数を確かめます。そのうえで、無人のとき・決めた時刻・告知して人を外したあとのどれかで反映し、最後に永続データが残っているかを確かめます。

Claude Code、Codex、Antigravity（`agy`）など、[Agent Skills](https://agentskills.io) の `SKILL.md` 形式に対応したエージェントで使えます。

## 入れるとどうなるか

- **反映のたびの「誰かいる？」がなくなります。** プレイヤー一覧やVCを開いて確かめる代わりに、エージェントがサーバーへ実際に問い合わせます。
- **待たずに反映できます。** 「誰もいなくなったら」と頼んで別の作業をすれば、裏で数え続け、無人になった瞬間に反映します。
- **時刻を決めて寝られます。** 「pm10:20に反映して、再起動OK」と頼むと、時刻まで待って直前に数え、まだ人がいれば告知して人を外してから反映します。
- **取得に失敗しても「無人」と思い込みません。** 数えられなければ止まります。
- **反映の証拠が残ります。** 反映した時点で何人いたか、どう数えたか、データが残っているかを報告します。

## こんなときに効きます

- **Minecraftサーバー。** 建築中の人を落とさずにプラグインや設定を更新できます。再起動の前には保存します。
- **Discordのボイス・読み上げ・音楽Bot。** 通話を切らずに新しい版へ入れ替えられます。
- **Terraria・Rust・ARK・CS2などのゲームサーバー。** パッチの再起動を、レイドの最中ではなく無人のときに回せます。
- **WebSocket・チャットなどのリアルタイムアプリ。** 開いているソケットがないときに入れ替えられます。
- **自前運用のWebアプリ。** アクセスログやセッションテーブルから、空いている時間を選べます。

静的サイトやサーバーレス関数のように、誰もつながり続けないサービスには向きません。数える対象がないためです。

## やること

1. 再読み込みで済む変更なら、止めずに反映します。
2. 人数の数え方を選び、1回試して動くことを確かめます。取得に失敗した場合を「0人」とは読みません。
3. 頼み方に合わせて反映します。
   - 「誰もいなければ」: 1回数えて、0人のときだけ反映
   - 「誰もいなくなったら」: バックグラウンドで数え続け、0人になったら反映（締め切りあり）
   - 「pm5:00に」: 時刻まで待ち、直前に数える
   - 「再起動OK」: 告知 → 入場を止める → 人を外す → 動いているうちに保存 → 反映
4. 起動したか、意図した版か、データが残っているか、入場制限を外したかを確かめます。
5. 反映した時点で何人いたかを、数えた方法と一緒に報告します。

## 対応している数え方

| サービス | 方法 |
|---|---|
| Minecraft（Java版、Geyser経由の統合版） | RCONの`list` |
| Terraria | コンソールの`playing`、TShockのREST `/v2/players/list` |
| Rust・ARK・CS2などSteam系 | A2Sクエリ、RustのWebRCON |
| Discordのボイス・読み上げ・音楽Bot | `GET /guilds/{id}/voice-states/@me` とBot自身の状態 |
| WebSocket・SSE・TCPのアプリ | `ss`で確立済みの接続数 |
| Webアプリ | 直近のアクセスログのIP、セッションテーブル |

詳しくは [skills/headcount/references/presence.md](skills/headcount/references/presence.md) を見てください。

## 入れ方

### 全エージェントにまとめて

```bash
npx skills add spa77k/headcount -g
```

[skills CLI](https://github.com/vercel-labs/skills) を使います。どのエージェントに入れるかを聞かれます。

### 手で入れる

```bash
git clone https://github.com/spa77k/headcount.git
```

`skills/headcount` を、各エージェントのスキルフォルダにコピーするか、シンボリックリンクを張ります。

| エージェント | 全プロジェクト共通 | プロジェクトごと |
|---|---|---|
| Claude Code | `~/.claude/skills/headcount` | `.claude/skills/headcount` |
| Codex | `~/.agents/skills/headcount` | `.agents/skills/headcount` |
| Antigravity CLI（`agy`） | `~/.gemini/antigravity-cli/skills/headcount` | `.agents/skills/headcount` |
| Antigravity IDE | `~/.gemini/config/skills/headcount` | `.agents/skills/headcount` |

## 使い方

次のように頼むと動きます。

```
誰もいなければ本番に反映して
pm10:20に反映して、再起動OK
VCに誰もいなくなったらBotを更新して
```

直接呼ぶこともできます: `/headcount 誰もいなければ反映`

デプロイの手順はスキルが勝手に作りません。SSH先、再起動スクリプト、コンテナ名などは `AGENTS.md` か `CLAUDE.md` に書いておいてください。スキルはそれに従います。

## ライセンス

MIT
