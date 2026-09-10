# jira-harness (Codex)

[`jira-harness`](https://github.com/bigbulgogiburger/jira-harness) v3.4.0 의 **Codex CLI 판**입니다.
저장소 루트가 곧 Codex 마켓플레이스이고, 그 안에 플러그인 하나가 들어 있습니다.

> ## 🔴 이 저장소는 생성물입니다 — 직접 고치지 마세요
>
> 원본 Claude Code 플러그인 트리에서 `build-codex.mjs` 가 **통째로 다시 만듭니다.**
> 여기서 고친 것은 다음 생성 때 사라집니다. 고칠 곳은 두 군데뿐입니다:
>
> - 원본 소스 — 로직·문서·스크립트
> - 원본의 `codex-overlay/` — Codex 에서만 달라야 하는 파일 (생성 시 위에 덮입니다)

## 설치

```bash
codex plugin marketplace add bigbulgogiburger/jira-harness_codex --ref main
codex plugin add jira-harness@jira-harness-codex
```

확인:

```bash
codex plugin list
codex debug prompt-input | grep "jira-harness:"   # 모델이 실제로 보는 스킬 목록
```

`codex debug prompt-input` 은 모델 호출 없이 **모델이 실제로 받는 입력**을 뽑습니다 —
"스킬이 등재됐나" 를 값싸게 확인하는 유일한 축입니다.

## 🔴 설치만으로는 훅이 발화하지 않습니다

이 플러그인은 `PreToolUse` 훅을 싣습니다. 그런데 **플러그인 설치와 훅 신뢰는 별개입니다.**
`~/.codex/config.toml` 의 `[hooks.state]` 에 그 정의의 `trusted_hash` 가 생겨야 실제로 돕니다.

실측: 신뢰 전에는 막혀야 할 `git commit` 이 **그냥 성공해서 커밋이 만들어집니다.**
이벤트 이름·`matcher`·`${PLUGIN_ROOT}` 는 원인이 아닙니다 — 그것들을 바꾸지 않고도
신뢰 조건에서는 정상 차단됐습니다.

**Codex CLI 에서 `/hooks` 로 이 플러그인의 훅 정의를 검토·신뢰하세요.**
그 전까지 게이트는 "있지만 아무것도 막지 않는" 상태입니다.

## 무엇이 들어 있나

| 경로 | 역할 |
|------|------|
| `.agents/plugins/marketplace.json` | 마켓플레이스 정의 — `codex plugin marketplace add` 가 읽는다 |
| `plugins/jira-harness/.codex-plugin/plugin.json` | 플러그인 매니페스트 (skills · subagents · hooks 인라인) |
| `plugins/jira-harness/skills/` | 스킬 6종 — Codex 가 `SKILL.md` 를 그대로 읽는다 |
| `plugins/jira-harness/subagents/` | 역할 원고 15종 (`*.toml`) — 아래 설명 참조 |
| `plugins/jira-harness/scripts/` | 상태 JSON · 게이트 · 기록 스크립트 (Node 20+) |
| `plugins/jira-harness/scripts/lanes-codex.mjs` | 리뷰 레인 실행기 — `codex exec` 로 레인을 띄운다 |
| `plugins/jira-harness/README.md` | 원본(Claude Code)의 README — 동작 설명은 유효, 설치법은 아니다 |

### 스킬

| 이름 | 설명 |
|------|------|
| `jira-harness:grill-me` | A relentless interview to sharpen a plan or design. |
| `jira-harness:grilling` | Grill the user relentlessly about a plan or design until reaching shared understanding. Use when the user want… |
| `jira-harness:issue` | Jira 이슈(ABC-123 꼴 키) 한 건 또는 여러 건의 개발 전 주기를 진행하는 라우터 — 브랜치·상태 JSON(start), 결정 인터뷰(grill), 계획 워크플로와 승인(plan), 구현… |
| `jira-harness:jira-create` | jira-create — 자연어 한 줄 또는 문서를 읽어 Jira 이슈를 생성합니다. 단일 이슈는 바로 등록하고, 문서 기반이면 에픽→이슈→하위이슈 계층으로 일괄 등록합니다. 'jira 이슈 만들어… |
| `jira-harness:kb-ingest` | kb-ingest — Jira 이슈 KEY 가 없는 소스 (회의록, 클라이언트 회신, Mattermost 스레드, 세션 진단·결정 기록, 외부 문서, 구두 전달 정보) 를 코드 프로젝트의 docs/… |
| `jira-harness:setup` | 프로젝트에 jira-harness v3 를 설치·점검·업그레이드하는 스킬 — 스택 자동 감지 → harness.json 작성(멱등) → 마켓플레이스/플러그인 등록 → 전제 체크리스트 → 위반 주입 … |

## Claude 판과 무엇이 다른가

| 축 | Claude Code | Codex |
|----|-------------|-------|
| 스킬 | `SKILL.md` | 같음 — 그대로 옮긴다 |
| 훅 | `hooks/hooks.json` | 매니페스트 인라인. 이벤트 이름은 같다. **신뢰 단계가 추가된다** |
| 서브에이전트 | 이름 붙은 정의를 이름으로 호출 | **없다.** `spawn_agent` 로 동질 에이전트를 띄우고 역할은 프롬프트로 준다 |
| 리뷰 레인 | Workflow 툴 (`workflows/*.js`) | `lanes-codex.mjs` + `codex exec` — 배정은 같은 glob 규칙으로 **스크립트가** 한다 |
| 스킬 호출 | `/plugin:skill` | 슬래시 문법이 없다. 이름만 `plugin:skill` 로 등재된다 |

서브에이전트가 "없다"는 것은 기능이 없다는 뜻이 아닙니다 — Codex 프롬프트가 직접
"팀의 모든 에이전트는 동등하게 유능하고 같은 툴을 쓴다"고 말합니다. 그래서 원본의
`agents/*.md` 15종은 버려지지 않고 `subagents/*.toml` 의 `developer_instructions` 로
옮겨져, 레인 실행기가 읽어 프롬프트에 끼웁니다.

## 아직 검증되지 않은 것

초록으로 보이는 것과 실제로 도는 것은 다른 말이라, 확인한 것만 확인했다고 적습니다.

- **리뷰 레인 실주행** — `lanes-codex.mjs` 는 구문과 "설정 없음" 경로만 확인했습니다.
  실제 LLM 레인이 완주해 finding 을 모으는 것은 미검증입니다.
- **`recon` / `plan` / `implement` 워크플로 미이식** — 문서 계약은 보존했지만 Codex 실행기를
  새로 만들지 않았습니다. 임의 구현으로 채우지 않았습니다.
- **`apply_patch` 계열 훅** — 파일 경로를 보는 훅은 Claude 의 `tool_name`/`file_path` 를
  가정합니다. Codex 는 `apply_patch` + `tool_input.command` 로 보내므로 무시될 수 있습니다(정적 확인).

## 다시 만들기

```bash
node scripts/build-codex.mjs --src <원본 플러그인 디렉토리> --out <이 저장소>
```

변환기는 [`caseworker`](https://github.com/bigbulgogiburger/caseworker) 의 `scripts/` 에 한 벌만
있고 두 플러그인을 모두 찍어냅니다. 치환 규칙은 **한 번도 발화하지 않으면 빌드가 exit 1 로 죽습니다** —
규칙을 넣어 놓고 안 먹는데 초록이면 그 산출물엔 그 검사가 0이기 때문입니다.

## 요구 사항

Codex CLI 0.153.4+ (`codex plugin` 서브커맨드), Node.js 20+, git. Windows 는 Git Bash.

## 라이선스

MIT — 원본과 같습니다.

<sub>생성 시각 2026-09-10T07:48:51.754Z · 원본 v3.4.0</sub>
