# emile-masterplan

[Claude Code](https://claude.com/claude-code) skill for tracking large, multi-plan bodies of work across sessions.

여러 개의 plan으로 나뉘는 큰 작업 묶음에 대해, 세션이 바뀌어도 흐름이 끊기지 않도록 **고정 진입점** 문서(마스터플랜)를 만들고 유지하는 Claude Code 스킬입니다.

마스터플랜은 **지도이지 설계가 아닙니다.** 설계 결정은 각 슬라이스의 spec이, 구현 절차는 각 슬라이스의 plan이 소유합니다. 이 문서는 이전 세션 기억이 없는 새 세션이 "지금 어디까지 왔고 다음에 무엇을 해야 하는가"를 복원하는 용도만 갖습니다.

## superpowers 워크플로에서의 위치

이 스킬은 [obra/superpowers](https://github.com/obra/superpowers)의 워크플로를 전제로 설계되었으며, 그 스킬 세트를 대체하지 않습니다.

```
brainstorming ──(범위가 plan 1개를 넘음)──> emile-masterplan ──> writing-plans (슬라이스 1개)
                                                  ↑                          │
                                                  └── 슬라이스 종료 시 갱신 ──┘
```

- 설계(spec)는 `superpowers:brainstorming`이 소유합니다.
- 슬라이스 plan은 `superpowers:writing-plans`가 소유합니다.
- 실행은 `superpowers:subagent-driven-development` 또는 `superpowers:executing-plans`가 소유합니다.
- 이 스킬은 그 사이를 잇는 **슬라이스 단위 장기 기억**만 소유합니다.

superpowers 없이도 스킬 자체는 동작하지만, 위 워크플로 스킬들이 함께 있을 때를 기준으로 설계되었습니다.

## When it triggers

- brainstorming/writing-plans가 "범위가 커서 여러 plan으로 나눠야 한다"고 판단했을 때
- 이미 나눠 놓은 plan들을 새 세션에서 이어서 진행할 때
- 슬라이스 하나를 끝내고 진행 상황을 기록해야 할 때
- 사용자가 "마스터플랜", "master plan", "전체 로드맵" 등을 요청할 때

**쓰지 않을 때:** plan 1개로 끝나는 작업. 슬라이스가 하나뿐이면 마스터플랜은 순수 오버헤드입니다 — 그냥 writing-plans로 갑니다.

## Modes

호출될 때마다 어떤 파일을 쓰기 전에 먼저 상태를 판별합니다.

| 판별 결과 | 모드 |
|---|---|
| 사용자가 인자로 파일/주제를 지정 | 지정 우선 |
| `docs/masterplans/` 후보 0개 | **NEW** — 새로 작성 |
| 브랜치로 좁힌 후보 1개 | **RESUME** — 이어서 진행 |
| 후보 2개 이상 | **정지** — 목록을 보여주고 사용자 지정을 기다림 |

소유 문서가 확정된 뒤 슬라이스를 하나 끝냈거나 갱신을 요청받으면 **UPDATE** 모드로 진행 상황표를 갱신합니다.

## What it produces

산출물은 **호출된 세션의 작업 리포** 기준 `docs/masterplans/{YYYYMMDD}-{HHMM}-{제목}-master-plan.md`입니다. (스킬 디렉터리 자체에는 절대 쓰지 않습니다.)

세션이 워크트리에서 작업 중이었다면 `docs/masterplans/`를 그 워크트리에 둘 수 있습니다. 어느 쪽에 둘지 애매하면 스킬은 진행을 멈추고 사용자에게 확인받습니다.

템플릿은 필수 9개 섹션 + 조건부 섹션으로 구성됩니다.

- 목적/배경, 왜 나눴는가
- 세션 연속성 프로토콜 (새 세션 시작 시 / 슬라이스 착수 시 / 슬라이스 종료 시 순서 고정)
- 슬라이스 목록, 순서와 의존
- 진행 상황표 (증거 없는 ✅ 금지 — 커밋 해시·게이트 출력·날짜 필수)
- 결정/가정 (D-번호로 기록, 뒤집혀도 삭제하지 않고 개정 사유와 함께 보존)
- 조건부: 요구사항 추적, 작업 환경, 게이트, 미설계 슬라이스 질문, 참조·선례, 관찰된 불일치

조건부 섹션은 해당 조건이 리포에서 실제로 관측될 때만(`CLAUDE.md`, `.claude/rules/`, `package.json`, 요구사항 베이스라인 파일 등) 삽입됩니다. 설계 근거는 [`DESIGN-NOTES.md`](./DESIGN-NOTES.md)에 있습니다.

## Install

Claude Code는 `~/.claude/skills/<name>/`(전역) 또는 `<repo>/.claude/skills/<name>/`(프로젝트 한정) 아래의 `SKILL.md`를 스킬로 인식합니다.

```bash
git clone https://github.com/emile-popcornsar/emile-masterplan.git ~/.claude/skills/emile-masterplan
```

또는 특정 프로젝트에서만 쓰려면 해당 리포의 `.claude/skills/emile-masterplan/`에 두면 됩니다.

## Usage

Claude Code 세션에서:

```
/emile-masterplan
```

또는 자연어로 "마스터플랜 만들어줘", "전체 로드맵 확인해줘"라고 요청하면 스킬 설명(description)을 보고 자동으로 매칭됩니다.

## Related

- [emile-handoff](https://github.com/emile-popcornsar/emile-handoff) — 슬라이스 **내부**에서 세션이 끊길 때 쓰는 단기 기억 스킬. 마스터플랜은 슬라이스 단위 장기 기억, 핸드오프는 슬라이스 내부 단기 기억입니다.

## License

MIT
