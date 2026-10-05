# headcount

[English](README.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | **한국어**

배포하기 전에 지금 접속 중인 사람이 몇 명인지 먼저 세는 에이전트 스킬입니다.

사람들이 쓰고 있는 게임 서버나 음성 봇을 그대로 배포하면, 플레이 중이거나 통화 중인 사람이 끊기고 저장되지 않은 데이터가 사라질 수도 있습니다. `headcount`를 설치하면 코딩 에이전트가 먼저 실제 접속자 수를 확인합니다. 그런 다음 서비스가 비었을 때, 정해둔 시각에, 또는 공지하고 사람을 내보낸 뒤에만 배포합니다. 배포가 끝나면 영구 데이터가 그대로인지도 확인합니다.

Claude Code, Codex, Antigravity(`agy`), 그리고 [Agent Skills](https://agentskills.io)의 `SKILL.md` 형식을 지원하는 모든 에이전트에서 쓸 수 있습니다.

## 왜 쓰나요

- **배포할 때마다 "지금 누구 있어?" 하고 확인하지 않아도 됩니다.** 플레이어 목록이나 음성 채널을 직접 열어보는 대신, 에이전트가 서버에 실제로 질의합니다.
- **기다리지 않아도 됩니다.** "아무도 없어지면 배포해줘"라고 맡기고 다른 일을 하세요. 뒤에서 계속 세다가 비는 순간 배포합니다.
- **시각을 정해두고 푹 잘 수 있습니다.** "오후 10시 20분에 반영해줘, 재시작해도 돼"라고 하면 그 시각까지 기다렸다가 직전에 세고, 아직 사람이 있으면 공지하고 내보낸 뒤 배포합니다.
- **조회에 실패해도 "비었다"고 착각하지 않습니다.** 세지 못하면 멈춥니다. 0명으로 간주하지 않습니다.
- **증거가 남습니다.** 배포한 순간 몇 명이 접속해 있었는지, 어떻게 셌는지, 데이터가 남아 있는지를 보고합니다.

## 이럴 때 효과적입니다

- **마인크래프트 서버.** 건축 중인 사람을 끊지 않고 플러그인이나 설정을 업데이트할 수 있습니다. 재시작 전에는 저장합니다.
- **Discord 음성 / TTS / 음악 봇.** 통화를 끊지 않고 새 버전으로 교체할 수 있습니다.
- **Terraria, Rust, ARK, CS2 같은 게임 서버.** 패치 재시작을 레이드 도중이 아니라 서버가 비었을 때 할 수 있습니다.
- **WebSocket / 채팅 / 실시간 앱.** 열린 소켓이 없을 때 배포할 수 있습니다.
- **직접 운영하는 웹 앱.** 접속 로그나 세션 테이블에서 한가한 시간을 고를 수 있습니다.

정적 사이트나 서버리스 함수처럼 아무도 연결을 유지하지 않는 서비스에는 맞지 않습니다. 셀 대상이 없기 때문입니다.

## 하는 일

1. 다시 불러오기만으로 되는 변경은 멈추지 않고 적용합니다.
2. 접속자를 세는 방법을 고르고 한 번 실행해 동작을 확인합니다. 조회 실패를 "0명"으로 읽지 않습니다.
3. 요청한 표현에 맞춰 배포합니다.
   - "아무도 없으면": 한 번 세고 0명일 때만 배포
   - "아무도 없어지면": 백그라운드에서 계속 세다가 0명이 되면 배포(마감 시간 있음)
   - "오후 10시 20분에": 그 시각까지 기다렸다가 직전에 세기
   - "재시작해도 돼": 공지 → 신규 입장 차단 → 사람 내보내기 → 실행 중에 저장 → 배포
4. 서비스가 정상인지, 의도한 버전인지, 데이터가 남아 있는지, 입장 제한이 풀렸는지 확인합니다.
5. 배포 시점의 접속자 수와 센 방법을 보고합니다.

## 지원하는 세는 방법

| 서비스 | 방법 |
|---|---|
| 마인크래프트(자바 에디션, Geyser를 통한 베드락) | RCON `list` |
| Terraria | 콘솔 `playing`, TShock REST `/v2/players/list` |
| Rust, ARK, CS2 등 Steam 게임 | A2S 쿼리, Rust WebRCON |
| Discord 음성 / TTS / 음악 봇 | `GET /guilds/{id}/voice-states/@me` 와 봇 자신의 상태 |
| WebSocket / SSE / TCP 앱 | `ss`로 확립된 연결 수 |
| 웹 앱 | 최근 접속 로그의 IP, 세션 테이블 |

자세한 내용은 [skills/headcount/references/presence.md](skills/headcount/references/presence.md)를 보세요.

## 설치

### 모든 에이전트에 한 번에

```bash
npx skills add spa77k/headcount -g
```

[skills CLI](https://github.com/vercel-labs/skills)를 사용합니다. 어느 에이전트에 설치할지 물어봅니다.

### 직접 설치

```bash
git clone https://github.com/spa77k/headcount.git
```

그다음 `skills/headcount`를 에이전트의 스킬 폴더에 복사하거나 심볼릭 링크로 연결하세요.

| 에이전트 | 전역 | 프로젝트별 |
|---|---|---|
| Claude Code | `~/.claude/skills/headcount` | `.claude/skills/headcount` |
| Codex | `~/.agents/skills/headcount` | `.agents/skills/headcount` |
| Antigravity CLI(`agy`) | `~/.gemini/antigravity-cli/skills/headcount` | `.agents/skills/headcount` |
| Antigravity IDE | `~/.gemini/config/skills/headcount` | `.agents/skills/headcount` |

## 사용법

다음과 같이 요청하면 동작합니다.

```
서버에 아무도 없으면 이걸 배포해줘.
오후 10시 20분에 반영해줘. 재시작해도 돼.
음성 채널에 아무도 없어지면 봇 업데이트를 올려줘.
```

직접 호출할 수도 있습니다: `/headcount 아무도 없으면 배포`

이 스킬은 배포 절차를 마음대로 만들지 않습니다. SSH 호스트, 재시작 스크립트, 컨테이너 이름 등은 `AGENTS.md`나 `CLAUDE.md`에 적어두세요. 스킬이 그대로 따릅니다.

## 라이선스

MIT
