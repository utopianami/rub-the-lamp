# rub-the-lamp

AI 협업 개발 환경을 위한 도구 모음입니다.

---

## 도구 목록

| 도구 | 목적 | 가이드 |
|------|------|--------|
| **AI-LINK** | Claude ↔ Cursor 교차 검증 프로토콜 | [Setup Guide](AI-LINK%20Setup%20Guide.md) |
| **Spec Kit + Test Workflow** | 피처 개발 사이클 (스펙 → 테스트 → 구현) | [Setup Guide](Spec%20Kit%20Setup%20Guide.md) |
| **oh-my-claudecode** | Claude Code 다중 에이전트 오케스트레이션 | [GitHub](https://github.com/Yeachan-Heo/oh-my-claudecode) |
| **tmux + Agent Team** | 멀티 에이전트 병렬 실행 환경 | [Guide](tmux%20%2B%20Agent%20Team%20Guide.md) |

---

## AI-LINK

Claude(코딩)와 Cursor(리뷰) 간의 **교차 검증 프로토콜**. 역할을 분리해서 한쪽이 작성한 코드를 다른 모델이 blind spot과 edge case를 검증합니다. 5개 슬롯으로 병렬 작업도 가능.

> 상세 가이드: [AI-LINK Setup Guide.md](AI-LINK%20Setup%20Guide.md)

---

## Spec Kit + Test Workflow

GitHub [spec-kit](https://github.com/github/spec-kit)에 **테스트 워크플로우를 추가**한 확장 버전. 구현 전 테스트 계획(testplan)을 세우고, 구현 후 테스트 리뷰(testreview)로 검증하는 사이클을 추가했습니다.

```
specify → plan → tasks → testplan → implement → testreview
```

> 상세 가이드: [Spec Kit Setup Guide.md](Spec%20Kit%20Setup%20Guide.md)

---

## oh-my-claudecode

Claude Code를 위한 **다중 에이전트 오케스트레이션 플러그인**. 28개 전문 에이전트(architect, executor, designer 등)가 작업 의도를 감지해서 자동으로 위임합니다. Haiku/Sonnet/Opus 3단계 모델 자동 선택.

| 키워드 | 동작 |
|-------|------|
| `ralph` | 완료까지 지속 실행 |
| `ulw` | 최대 병렬 실행 |
| `autopilot` | 완전 자동 실행 |

> GitHub: [Yeachan-Heo/oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)

---

## tmux + Agent Team

tmux 터미널 멀티플렉서로 **여러 Claude 에이전트를 물리적으로 병렬 실행**. 독립 태스크를 패널별로 분배하고, 구현/리뷰/테스트 역할을 나눠서 동시 작업합니다. 터미널이 꺼져도 세션이 유지됩니다.

```
┌──────────────┬──────────────┐
│ Agent A      │ Agent B      │
│ config 구현   │ registry 구현 │
├──────────────┼──────────────┤
│ Agent C      │ Reviewer     │
│ metrics 구현  │ 리뷰 대기     │
└──────────────┴──────────────┘
```

> 상세 가이드: [tmux + Agent Team Guide.md](tmux%20%2B%20Agent%20Team%20Guide.md)

---

## 조합 가이드

도구들을 조합해서 사용하는 시나리오 모음입니다.

> **[Workflow Recipes.md](Workflow%20Recipes.md)** — 2개~4개 도구 조합 패턴과 실전 예시

| 상황 | 추천 조합 |
|-----|----------|
| 대규모 피처, 최대 병렬화 | Spec Kit + OMC + AI-LINK + tmux |
| 큰 피처, 문서화 필요 | Spec Kit + OMC + AI-LINK |
| 빠른 개발, 개인 작업 | OMC + AI-LINK |
| 간단한 수정 | AI-LINK만 |

---

## 설정 파일 구조

```
┌─────────────────────────────────────────────┐
│ ~/.claude/CLAUDE.md                         │ ← oh-my-claudecode
│   - ralph, ultrawork, planning 등           │
├─────────────────────────────────────────────┤
│ /project/CLAUDE.md + .cursorrules           │ ← AI-LINK
│   - 코드 작성 후 AI-LINK.md 업데이트        │
├─────────────────────────────────────────────┤
│ spec-kit (specify init)                     │ ← Spec Kit
│   - /speckit.specify, /speckit.plan 등      │
├─────────────────────────────────────────────┤
│ ~/.tmux.conf                                │ ← tmux + Agent Team
│   - gruvbox 테마, 패널 분할, 마우스 지원     │
└─────────────────────────────────────────────┘
```
