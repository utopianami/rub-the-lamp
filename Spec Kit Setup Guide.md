

이 파일을 Claude에게 보여주고 "이거 읽고 세팅 도와줘"라고 하세요.

---

## 전체 그림

```
┌─────────────────────────────────────────────────────────────┐
│                    초기 세팅 (1회)                           │
│                                                             │
│  ┌──────────┐  ┌──────────┐                                 │
│  │ Install  │→ │   Init   │  ← Claude가 도와줌              │
│  │ CLI 설치  │  │ 설정작성  │                                 │
│  └──────────┘  └──────────┘                                 │
│                     ↓                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Constitution │→ │   Context    │→ │  Test Workflow   │  │
│  │   (필수)      │  │   (선택)     │  │     (선택)       │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓
                        세팅 완료!
                              ↓
┌─────────────────────────────────────────────────────────────┐
│              피처 개발 사이클 (반복)                          │
│                                                             │
│    ┌─────────┐    ┌─────────┐    ┌─────────┐              │
│    │ specify │ →  │  plan   │ →  │  tasks  │              │
│    │ 스펙작성 │    │ 계획수립 │    │ 태스크  │              │
│    └─────────┘    └─────────┘    └─────────┘              │
│                                       ↓                    │
│    ┌─────────┐    ┌─────────┐    ┌─────────┐              │
│    │testreview│ ← │implement│ ← │testplan │              │
│    │테스트리뷰│    │  구현   │    │테스트계획│              │
│    └─────────┘    └─────────┘    └─────────┘              │
│         ↓                                                  │
│      완료! → 다음 피처 → specify부터 다시                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Install

```bash
# specify-cli 설치
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# 새 프로젝트 생성
specify init <PROJECT_NAME>

# 또는 기존 프로젝트에 초기화
specify init . --ai claude
```

**참고:** [GitHub Spec Kit](https://github.com/github/spec-kit)

---

## Init (Claude 실행 지침)

아래 순서대로 사용자와 대화하며 초기화를 진행하세요.

### Step 1: Constitution 설정 (필수)

사용자에게 물어보세요:
> "프로젝트에 대해 알려주세요:
> 1. 프로젝트가 뭔가요? (한 줄 설명)
> 2. 주요 기술 스택은? (언어, 프레임워크)
> 3. 가장 중요한 원칙 3가지는?"

응답 받으면 `.specify/memory/constitution.md` 생성:

```markdown
# Project Constitution

## Project Overview
[사용자 답변 1]

## Core Principles
1. [원칙 1]
2. [원칙 2]
3. [원칙 3]

## Tech Stack
[사용자 답변 2]
```

### Step 2: Context 설정 (선택)

사용자에게 물어보세요:
> "추가 컨텍스트 있나요? (디렉토리 구조, 데이터 경로 등)
> 없으면 '없음'"

'없음'이 아니면 `.specify/memory/context.md` 생성.

### Step 3: Test Workflow 추가 (선택)

사용자에게 물어보세요:
> "Test Workflow 추가할까요?
> - testplan: 구현 전 테스트 계획
> - testreview: 구현 후 테스트 리뷰
>
> ㅇㅇ / ㄴㄴ"

'ㅇㅇ' 응답 시:
→ 같은 폴더의 아래 `Spec Kit: Test Workflow 추가 가이드` 읽고 그대로 적용

### Step 4: 완료

> "세팅 완료! 이제 피처 개발을 시작하세요:
>
> `/speckit.specify '피처 설명'`"

---

## 피처 개발 사이클 요약

| 단계 | 명령어 | 하는 일 |
|-----|-------|--------|
| 1 | `/speckit.specify "설명"` | 피처 스펙 정의 |
| 2 | `/speckit.plan` | 구현 계획 수립 |
| 3 | `/speckit.tasks` | 태스크 목록 생성 |
| 4 | `/speckit.testplan "검증항목"` | 테스트 계획 (선택) |
| 5 | `/speckit.implement` | 구현 |
| 6 | `/speckit.testreview` | 테스트 리뷰 (선택) |

**간단한 피처는** 1→3→5 만으로도 충분합니다.

---

## Sources

- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [Spec Kit Releases](https://github.com/github/spec-kit/releases)






# Spec Kit: Test Workflow 추가 가이드

기존 Spec Kit에 test-plan / test-review 워크플로우를 추가하는 방법입니다.

## 워크플로우

```
기존: specify → plan → tasks → implement
추가: specify → plan → tasks → testplan → implement → testreview
```

---

## 1. 템플릿 파일 생성

### `.specify/templates/test-plan-template.md`

```markdown
---
description: "Pre-implementation test plan - define what to test before coding"
---

# Test Plan: [FEATURE NAME]

**Date**: [DATE]
**Feature**: [link to spec.md]
**Status**: implement 전 검증 기준 정의

---

## 사용자 지정 테스트

> 사용자가 명시적으로 요청한 검증 항목

### 필수 검증 (Must Have)

- [ ] [검증 항목 1]
- [ ] [검증 항목 2]

### 선택 검증 (Nice to Have)

- [ ] [검증 항목]

---

## AI 권장 테스트

> spec.md, plan.md 분석 결과 필요한 검증 항목

### 데이터 무결성

- [ ] [데이터 검증 항목]

### 기능 동작

- [ ] [기능 검증 항목]

### 엣지 케이스

- [ ] [경계 조건 검증]

### 에러 처리

- [ ] [에러 시나리오]

---

## 검증 방법

### 수동 테스트

\`\`\`bash
# 검증 명령어 예시
[command]
\`\`\`

**예상 결과:**
\`\`\`text
[성공 시 출력]
\`\`\`

### 자동 테스트 (필요시)

\`\`\`bash
# 자동화된 검증 스크립트
[command]
\`\`\`

---

## 수락 기준 (Acceptance Criteria)

| 항목 | 기준 | 검증 방법 |
|-----|-----|---------|
| [항목 1] | [기준] | [방법] |
| [항목 2] | [기준] | [방법] |

---

## Notes

- implement 완료 후 이 항목들을 기준으로 검증
- 놓친 케이스는 test-review에서 추가
```

---

### `.specify/templates/test-review-template.md`

```markdown
---
description: "Post-implementation test review - verify implementation and identify missing tests"
---

# Test Review: [FEATURE NAME]

**Date**: [DATE]
**Feature**: [link to spec.md]
**Test Plan**: [link to test-plan.md]
**Status**: implement 후 검증 및 리뷰

---

## Test Plan 검증 결과

> test-plan.md에서 정의한 항목 검증

### 필수 검증 결과

| 항목 | 결과 | 비고 |
|-----|-----|-----|
| [항목 1] | ✅ / ❌ | [비고] |
| [항목 2] | ✅ / ❌ | [비고] |

### 선택 검증 결과

| 항목 | 결과 | 비고 |
|-----|-----|-----|
| [항목] | ✅ / ❌ / ⏭️ 스킵 | [비고] |

---

## 추가 발견 사항

> 구현 과정에서 발견된 테스트 필요 항목

### 놓친 케이스

- [ ] [test-plan에서 놓친 케이스 1]
- [ ] [test-plan에서 놓친 케이스 2]

### 새로운 엣지 케이스

- [ ] [구현 중 발견된 엣지 케이스]

---

## 테스트 코드 필요 여부

> 자동화 테스트 코드 작성이 필요한 항목

### 필요함

| 테스트 유형 | 대상 | 이유 |
|-----------|-----|-----|
| [unit/integration/e2e] | [대상] | [이유] |

### 불필요함

| 항목 | 이유 |
|-----|-----|
| [항목] | [이유: 수동 검증으로 충분 등] |

---

## 테스트 실행 결과

### 명령어 및 결과

\`\`\`bash
# 실행한 테스트 명령어
[command]
\`\`\`

**출력:**
\`\`\`text
[실제 출력 결과]
\`\`\`

### 최종 상태

- **전체 테스트**: ✅ 통과 / ❌ 실패
- **커버리지**: [해당시 %]

---

## Action Items

> 추가로 필요한 작업

- [ ] [추가 테스트 코드 작성]
- [ ] [문서 업데이트]
- [ ] [버그 수정]

---

## Notes

- [테스트 관련 참고사항]
```

---

## 2. 명령어 파일 생성

### `.claude/commands/speckit.testplan.md`

```markdown
---
description: Create pre-implementation test plan defining what to verify before coding.
handoffs:
  - label: Implement Project
    agent: speckit.implement
    prompt: Start the implementation
    send: true
---

## User Input

\`\`\`text
$ARGUMENTS
\`\`\`

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `.specify/scripts/bash/check-prerequisites.sh --json` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute.

2. **Load context**: Read from FEATURE_DIR:
   - **Required**: spec.md (user stories, requirements)
   - **Required**: plan.md (tech stack, architecture)
   - **Optional**: tasks.md (implementation tasks)
   - **Optional**: data-model.md (entities)
   - **Optional**: contracts/ (API specs)

3. **사용자 지정 테스트 수집**:
   - $ARGUMENTS에서 사용자가 명시한 테스트 요구사항 추출
   - 없으면 사용자에게 질문: "이 기능에서 꼭 검증하고 싶은 항목이 있나요?"

4. **AI 권장 테스트 생성**:
   - spec.md의 user stories 분석 → 각 스토리별 검증 항목
   - plan.md의 기술 스택 분석 → 데이터 무결성 검증 항목
   - data-model.md 분석 → 엔티티 관계 검증 항목
   - contracts/ 분석 → API 응답 검증 항목
   - 일반적인 엣지 케이스 및 에러 시나리오 추가

5. **검증 방법 작성**:
   - 각 테스트 항목에 대한 구체적인 검증 명령어
   - 예상 결과 (성공/실패 케이스)
   - 수동 vs 자동 테스트 구분

6. **Generate test-plan.md**: Use `.specify/templates/test-plan-template.md` as structure, fill with:
   - 사용자 지정 테스트 (필수/선택 구분)
   - AI 권장 테스트 (카테고리별)
   - 검증 방법 및 명령어
   - 수락 기준 테이블

7. **Report**: Output path to generated test-plan.md and summary:
   - 총 테스트 항목 수
   - 필수 vs 선택 항목 수
   - 검증 방법 유형 (수동/자동)

Context for test plan: $ARGUMENTS
```

---

### `.claude/commands/speckit.testreview.md`

```markdown
---
description: Post-implementation test review - verify implementation and identify missing tests or test code needs.
---

## User Input

\`\`\`text
$ARGUMENTS
\`\`\`

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `.specify/scripts/bash/check-prerequisites.sh --json` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute.

2. **Load context**: Read from FEATURE_DIR:
   - **Required**: test-plan.md (pre-implementation test plan)
   - **Required**: tasks.md (implementation tasks - check completed items)
   - **Optional**: spec.md, plan.md (for reference)

3. **구현 코드 분석**:
   - tasks.md에서 완료된 태스크 확인
   - 구현된 파일들 읽기 (tasks.md의 파일 경로 참조)
   - 코드 구조 및 로직 파악

4. **Test Plan 검증**:
   - test-plan.md의 각 항목을 구현된 코드와 대조
   - 각 항목별 검증 실행 (가능한 경우)
   - 결과 기록: ✅ 통과, ❌ 실패, ⏭️ 스킵

5. **추가 발견 사항 분석**:
   - test-plan.md에서 놓친 케이스 식별
   - 구현 과정에서 발생한 새로운 엣지 케이스
   - 코드 리뷰 관점에서 테스트가 필요한 부분

6. **테스트 코드 필요 여부 판단**:
   - 자동화 테스트가 필요한 항목 식별:
     - 반복 실행이 필요한 검증
     - 회귀 테스트로 유지해야 할 항목
     - CI/CD에 포함해야 할 검증
   - 불필요한 항목:
     - 수동 검증으로 충분한 경우
     - 일회성 검증인 경우

7. **테스트 실행**:
   - test-plan.md의 검증 명령어 실행
   - 결과 캡처 및 기록

8. **Generate test-review.md**: Use `.specify/templates/test-review-template.md` as structure, fill with:
   - Test Plan 검증 결과 테이블
   - 추가 발견 사항 목록
   - 테스트 코드 필요 여부 분석
   - 테스트 실행 결과
   - Action Items

9. **Report**: Output summary:
   - 테스트 통과/실패 현황
   - 추가 필요한 테스트 코드 목록
   - Action Items 요약

Context for test review: $ARGUMENTS
```

---

## 3. 기존 명령어 수정

### `.claude/commands/speckit.tasks.md`

**handoffs 섹션에 추가:**

```yaml
handoffs:
  - label: Analyze For Consistency
    agent: speckit.analyze
    prompt: Run a project analysis for consistency
    send: true
  - label: Create Test Plan          # 이 부분 추가
    agent: speckit.testplan
    prompt: Define what to test before implementation
    send: true
  - label: Implement Project
    agent: speckit.implement
    prompt: Start the implementation in phases
    send: true
```

---

### `.claude/commands/speckit.implement.md`

**handoffs 섹션 추가 (기존에 없으면 생성):**

```yaml
---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md
handoffs:                            # 이 부분 추가
  - label: Test Review
    agent: speckit.testreview
    prompt: Review implementation and identify missing tests
    send: true
---
```

---

## 4. (선택) 경로 설정 변경

`.specify`를 서브디렉토리(예: `research/`)에 두고 싶으면 `common.sh`의 `get_repo_root()` 함수 수정:

```bash
get_repo_root() {
    local script_dir="$(CDPATH="" cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

    # .specify 폴더 위치를 프로젝트 루트로 사용
    local specify_root="$(cd "$script_dir/../../.." && pwd)"
    if [[ -d "$specify_root/.specify" ]]; then
        echo "$specify_root"
        return
    fi

    # Fallback to git root
    if git rev-parse --show-toplevel >/dev/null 2>&1; then
        git rev-parse --show-toplevel
    else
        (cd "$script_dir/../../.." && pwd)
    fi
}
```

---

## 사용법

```bash
# 기존 워크플로우
/speckit.specify "피처 설명"
/speckit.plan
/speckit.tasks

# 테스트 계획 (implement 전)
/speckit.testplan "검증하고 싶은 항목"

# 구현
/speckit.implement

# 테스트 리뷰 (implement 후)
/speckit.testreview
```
