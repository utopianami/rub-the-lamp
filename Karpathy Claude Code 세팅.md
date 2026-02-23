

## 1. CLAUDE.md (글로벌)

Karpathy 4원칙이 담긴 행동 지침서. 모든 프로젝트에서 항상 읽힘.

```bash
curl -o ~/.claude/CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md
```

## 2. Plugin (글로벌)

마켓플레이스 추가 + 플러그인 설치. Skills 자동 로드.

```bash
# 마켓플레이스 추가
claude plugin marketplace add forrestchang/andrej-karpathy-skills

# 플러그인 설치
claude plugin install andrej-karpathy-skills@karpathy-skills -s user

# 임시 폴더 정리
rm -rf /tmp/andrej-karpathy-skills
```

## 확인

```bash
claude plugin list
```

## 개념 정리

|구분|역할|적용 방식|
|---|---|---|
|`CLAUDE.md`|항상 읽히는 고정 규칙|매 대화마다 자동 로드|
|`Plugin`|맥락에 따른 전문 지침 (Skills)|작업 맥락에 맞게 자동 로드|

## 참고 링크

- Karpathy 원문 트윗: https://x.com/karpathy/status/2015883857489522876
- GitHub 레포: https://github.com/forrestchang/andrej-karpathy-skills

## Karpathy 4원칙

1. **Think Before Coding** - 가정하지 말고, 모호하면 물어봐라
2. **Simplicity First** - 최소한의 코드, 과도한 추상화 금지
3. **Surgical Changes** - 요청한 것만 건드려라
4. **Goal-Driven Execution** - "해줘" 대신 "성공 기준"을 줘라

## Karpathy 트윗 번역 (원문: https://x.com/karpathy/status/2015883857489522876)

### 코딩 워크플로우

최근 LLM 코딩 능력의 급격한 향상으로, 11월엔 수동+자동완성 80% / 에이전트 20%였던 작업이 12월에는 에이전트 80% / 수정·마무리 20%로 바뀌었다. 즉, 지금 나는 거의 영어로 코딩하고 있다. 자존심이 좀 상하지만, 큰 "코드 액션" 단위로 소프트웨어를 다루는 힘이 너무 유용하다. 약 20년 개발 경력에서 기본 코딩 워크플로우가 이렇게 바뀐 건 처음이고, 불과 몇 주 만에 일어났다.

### 문제점들 (CLAUDE.md로 고치려 했지만 잘 안 됨)

- 모델은 당신 대신 잘못된 가정을 하고 그냥 달려간다. 혼란스러움을 관리하지 않고, 명확화를 요청하지 않고, 불일치를 드러내지 않고, 트레이드오프를 제시하지 않고, 해야 할 때 반박하지 않는다.
- 코드와 API를 과도하게 복잡하게 만들고 싶어한다. 추상화를 부풀리고, 죽은 코드를 정리하지 않고, 100줄이면 될 걸 1000줄짜리 거대한 구조물로 구현한다.
- 자신이 충분히 이해하지 못한 주석과 코드를 부작용으로 변경/삭제하는 경우가 여전히 있다.

### 레버리지 (핵심)

LLM은 구체적인 목표를 달성할 때까지 루프를 도는 것에 탁월하다. **무엇을 하라고 시키지 말고, 성공 기준을 주고 알아서 하게 해라.** 테스트를 먼저 작성하고 통과시키게 해라.

### 끈질김 (Tenacity)

에이전트가 무언가를 집요하게 파고드는 걸 지켜보는 게 정말 흥미롭다. 지치지 않고, 낙담하지 않고, 사람이라면 포기할 상황에서도 계속 시도한다. 체력(stamina)이 작업의 핵심 병목이었는데, LLM으로 그게 극적으로 늘었다.

### 능력 퇴화 (Atrophy)

수동으로 코드를 작성하는 능력이 서서히 퇴화하고 있다는 걸 느끼기 시작했다. 코드 생성(쓰기)과 코드 판별(읽기)은 뇌에서 다른 능력이다.