# rub-the-lamp

AI 협업 개발 환경을 위한 도구 모음입니다.

---

## 도구 목록

| 도구 | 목적 | 사용 시나리오 |
|------|------|--------------|
| **AI-LINK** | Claude ↔ Cursor 협업 프로토콜 | 코드 작성 + 리뷰 교차 검증 |
| **Spec Kit + Test Workflow** | 피처 개발 사이클 관리 | 스펙 → 계획 → 테스트 계획 → 구현 → 테스트 리뷰 |
| **oh-my-claudecode** | Claude Code 다중 에이전트 | 병렬 처리로 구현 가속 |

---

## AI-LINK

Claude(코딩)와 Cursor(리뷰) 간의 협업 프로토콜입니다.

### 핵심 개념

- **확증 편향 방지**: 동일 모델의 blind spot을 다른 모델이 검증
- **교차 검증**: 서로 다른 추론 경로로 edge case 탐색

### 협업 사이클

```
Claude 코드 작성 → Cursor 리뷰 → [bad/neutral] → Claude 수정 → 반복
                              → [good] → 완료
```

### 세팅

1. `CLAUDE.md` - Claude 지침 (코딩 역할)
2. `.cursorrules` - Cursor 지침 (리뷰 역할)
3. `AI-LINK.md` - 공유 상태 관리 (5개 슬롯)

### 사용법

```bash
# Claude에게: 코딩 요청
"이거 구현해줘: [기능 설명]"

# Cursor에게: 리뷰 요청
"AI-LINK.md 확인하고 리뷰해줘"

# Claude에게: 피드백 반영
"피드백 처리해줘"
```

> 상세 가이드: [AI-LINK Setup Guide.md](AI-LINK%20Setup%20Guide.md)

---

## Spec Kit + Test Workflow

GitHub [spec-kit](https://github.com/github/spec-kit)에 **테스트 워크플로우를 추가**한 확장 버전입니다.

### 기존 spec-kit과의 차이점

```
기존 spec-kit:  specify → plan → tasks → implement

이 버전:        specify → plan → tasks → testplan → implement → testreview
                                              ↑                    ↑
                                         구현 전 테스트 계획    구현 후 테스트 리뷰
```

### 왜 테스트 워크플로우를 추가했나?

| 단계 | 목적 |
|-----|------|
| **testplan** | 구현 **전**에 검증 기준 정의 → 놓치는 케이스 방지 |
| **testreview** | 구현 **후** 테스트 계획 대비 검증 + 추가 엣지케이스 발견 |

### 설치

```bash
# specify-cli 설치
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# 프로젝트 초기화
specify init <PROJECT_NAME>
```

### 초기 세팅 (Claude에게 요청)

1. **Constitution 설정** (필수): 프로젝트 설명, 기술 스택, 핵심 원칙
2. **Context 설정** (선택): 추가 컨텍스트
3. **Test Workflow 설정** (권장): testplan/testreview 템플릿 및 명령어 추가

### 사용법

| 명령어 | 하는 일 | 비고 |
|-------|--------|------|
| `/speckit.specify "설명"` | 피처 스펙 정의 | |
| `/speckit.plan` | 구현 계획 수립 | |
| `/speckit.tasks` | 태스크 목록 생성 | |
| `/speckit.testplan "검증항목"` | 테스트 계획 | **추가됨** |
| `/speckit.implement` | 구현 | |
| `/speckit.testreview` | 테스트 리뷰 | **추가됨** |

> 상세 가이드: [Spec Kit Setup Guide.md](Spec%20Kit%20Setup%20Guide.md)

---

## oh-my-claudecode

Claude Code를 위한 **다중 에이전트 오케스트레이션 플러그인**입니다.

### 핵심 기능

- **28개 전문 에이전트**: 아키텍트, 연구자, 코더 등이 작업 의도 자동 감지
- **병렬 처리**: 복잡한 작업을 동시에 처리
- **데이터 분석**: Haiku/Sonnet/Opus 기반 과학자 에이전트

### 키워드

| 키워드 | 동작 |
|-------|------|
| `ralph` | 완료까지 지속 실행 |
| `ulw` | 최대 병렬 실행 |
| `plan` | 계획 수립 인터뷰 |
| `autopilot` | 완전 자동 실행 |

### 설치

```bash
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode
/oh-my-claudecode:omc-setup
```

> GitHub: [Yeachan-Heo/oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)

---

## 조합 시나리오

### 각 도구의 역할

| 도구 | 담당 | 강점 |
|-----|------|------|
| **Spec Kit** | 기획 + 최종 검증 | 체계적인 문서화, 테스트 계획 |
| **oh-my-claudecode** | 구현 가속 | 병렬 처리, 전문 에이전트 자동 배치 |
| **AI-LINK** | 품질 검증 | 다른 모델(Cursor)의 blind spot 검증 |

---

### 설정 파일 구조

```
┌─────────────────────────────────────────────┐
│ ~/.claude/CLAUDE.md (OMC)                   │ ← 워크플로우 자동화
│   - ralph, ultrawork, planning 등           │
├─────────────────────────────────────────────┤
│ /project/CLAUDE.md (AI-LINK)                │ ← 교차 검증 프로토콜
│   - 코드 작성 후 AI-LINK.md 업데이트        │
├─────────────────────────────────────────────┤
│ spec-kit                                    │ ← 기능 스펙 관리
│   - /speckit.specify, /speckit.plan 등      │
└─────────────────────────────────────────────┘
```

### 실제 사용 예시

```bash
# 1. 기능 스펙 작성 (spec-kit)
/speckit.specify "새 거래 알고리즘"

# 2. 코딩 시작 (OMC + AI-LINK)
"ralph: 거래 알고리즘 구현해줘"
→ OMC가 끝날 때까지 지속
→ 코드 완료 시 AI-LINK.md 자동 업데이트

# 3. Cursor로 전환하여 리뷰
→ AI-LINK.md의 "[x] 리뷰중 (Cursor)" 슬롯 처리
```

---

### 3개 조합: Spec Kit + oh-my-claudecode + AI-LINK

체계적인 문서화 + 빠른 구현 + 외부 검증이 모두 필요할 때

```
┌─────────────────────────────────────────────────────────────────┐
│  Phase 1: 기획 (Spec Kit)                                        │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌──────────┐     │
│  │ specify │ →  │  plan   │ →  │  tasks  │ →  │ testplan │     │
│  └─────────┘    └─────────┘    └─────────┘    └──────────┘     │
└─────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────┐
│  Phase 2: 구현 (oh-my-claudecode)                                │
│                                                                  │
│  "implement ralph ulw" → 28개 에이전트가 병렬로 빠르게 구현        │
└─────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────┐
│  Phase 3: 교차 검증 (AI-LINK)                                    │
│                                                                  │
│  Claude → AI-LINK.md 업데이트 → Cursor 리뷰                      │
│     ↑                               │                            │
│     └────── [bad/neutral] ──────────┘                            │
│              [good] → 다음 단계                                   │
└─────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────┐
│  Phase 4: 최종 검증 (Spec Kit)                                   │
│  ┌────────────┐                                                  │
│  │ testreview │ → testplan 대비 검증 + 엣지케이스 확인            │
│  └────────────┘                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**사용 예시**:
```bash
# 1. 스펙 정의
/speckit.specify "사용자 인증 기능"
/speckit.plan
/speckit.tasks
/speckit.testplan "JWT 만료 처리, 비밀번호 해싱"

# 2. 구현
"implement ralph ulw"

# 3. 교차 검증
# → Claude가 AI-LINK.md 업데이트
# → Cursor에게 "리뷰해줘"
# → [good] 될 때까지 반복

# 4. 최종 검증
/speckit.testreview
```

---

### 2개 조합: oh-my-claudecode + AI-LINK

Spec Kit 없이 가볍게 사용. 빠른 개발, 개인 작업에 적합.

```
┌─────────────────────────────────────────────────────────────────┐
│  구현 (oh-my-claudecode)                                         │
│                                                                  │
│  "구현해줘 ralph ulw" → 28개 에이전트가 병렬로 빠르게 구현         │
└─────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────┐
│  검증 (AI-LINK)                                                  │
│                                                                  │
│  Claude → AI-LINK.md 업데이트 → Cursor 리뷰                      │
│     ↑                               │                            │
│     └────── [bad/neutral] ──────────┘                            │
│              [good] → 완료                                       │
└─────────────────────────────────────────────────────────────────┘
```

**사용 예시**:
```bash
# 1. 구현
"로그인 폼 구현해줘 ralph ulw"

# 2. 검증
# → Claude가 AI-LINK.md 업데이트
# → Cursor에게 "리뷰해줘"
# → [good] 될 때까지 반복
```

**장점**:
- 문서화 오버헤드 없음
- 빠른 피드백 루프
- 외부 모델(Cursor)의 시선으로 검증

---

## 언제 어떤 조합을 쓸까?

| 상황 | 추천 조합 |
|-----|----------|
| 큰 피처, 팀 협업, 문서화 필요 | **3개 조합** (Spec Kit + omc + AI-LINK) |
| 빠른 개발, 개인 작업 | **2개 조합** (omc + AI-LINK) |
| 간단한 수정, 버그 픽스 | **AI-LINK만** |
| 체계적 기획만 필요 (검증은 자체) | **Spec Kit만** |
