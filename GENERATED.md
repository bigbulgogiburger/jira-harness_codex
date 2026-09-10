# 이 저장소는 생성물이다 — 직접 고치지 말 것

`jira-harness` v3.4.0 의 Codex 판. 원본은 Claude Code 플러그인이고,
이 트리는 그 원본에서 `scripts/build-codex.mjs` 가 통째로 다시 만든다.

여기서 고친 것은 **다음 생성 때 사라진다.** 고칠 곳은 두 군데뿐이다:

- 원본 소스 — 로직·문서·스크립트
- 원본의 `codex-overlay/` — Codex 에서만 달라야 하는 파일

## 다시 만들기

```
node scripts/build-codex.mjs --src <원본> --out <이 저장소>
```

## 설치

```
codex plugin marketplace add bigbulgogiburger/jira-harness_codex
codex plugin add jira-harness@jira-harness-codex
```

설치 후 Codex CLI의 `/hooks`에서 플러그인 훅 정의를 검토하고 신뢰해야 한다. 설치만으로 훅이 발화하지 않는다.
검증 방법과 포팅 범위: [Codex 안내](plugins/jira-harness/docs/codex-validation.md).

생성 시각: 2026-09-10T07:07:24.311Z
