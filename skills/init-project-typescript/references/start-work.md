---
description: "TDD/SDV 워크플로우 - 병렬 리서치 → 시나리오 작성 → 사용자 승인 → 병렬 구현 → GC → 검증"
agent: atlas
subtask: true
---

# /start-work: $ARGUMENTS

## 사전 확인

!git status
!git log --oneline -5
!ls .opencode/test-scenarios/ 2>/dev/null && echo "기존 시나리오 있음" || echo "시나리오 없음 (새로 작성 필요)"

@.sisyphus/plans/

---

## 실행 지침 (6단계 순서 엄수)

---

### ⬛ PHASE 1: 병렬 리서치

아래 에이전트 2개를 **동시에** 백그라운드로 실행하세요.
**두 결과가 모두 돌아올 때까지 PHASE 2로 넘어가지 마세요.**

**코드베이스 탐색** — explore 에이전트

```
delegate_task(
  subagent_type='explore',
  run_in_background=True,
  prompt='''
    CONTEXT: $ARGUMENTS 구현을 위한 코드베이스 탐색
    TASK:
      1. 작업 범위와 관련된 소스 파일 목록 (전체 경로 + 핵심 함수·클래스 시그니처)
      2. 기존 테스트 파일 위치 및 테스트 프레임워크 (Jest/Vitest 등)
      3. 재사용 가능한 유사 구현 패턴 및 모킹/픽스처 패턴
      4. 수정 시 영향받는 의존 모듈
      5. .opencode/test-scenarios/ 내 기존 시나리오 파일
      6. prisma/schema.prisma 변경 필요 여부 판단
    OUTPUT: 구조화된 탐색 보고서 (파일 경로 필수 포함)
  '''
)
```

**라이브러리/문서 조사** — librarian 에이전트

```
delegate_task(
  subagent_type='librarian',
  run_in_background=True,
  prompt='''
    CONTEXT: $ARGUMENTS 구현에 필요한 라이브러리/API 조사
    TASK:
      1. 관련 라이브러리 공식 API 및 버전
      2. 테스트 유틸리티 / 모킹 라이브러리 사용법
      3. 알려진 버그·제한사항·엣지 케이스
      4. TDD 맥락에서 권장되는 모킹/픽스처 전략
    OUTPUT: 라이브러리 조사 보고서 (Edge Cases 도출에 활용 가능한 형식)
  '''
)
```

필요한 경우 추가 실행 (선택):
- websearch MCP: 최신 베스트 프랙티스, 보안 이슈 검색
- grep_app MCP: 유사 오픈소스 구현 패턴 검색
- context7 MCP: 공식 문서 최신 버전 확인

---

### ⬛ PHASE 2: TDD/SDV 테스트 시나리오 + 테스트 파일 작성

PHASE 1 결과를 반영하여 아래 두 가지를 생성하세요.

#### 2-A. 시나리오 파일 생성

`.opencode/test-scenarios/{task-kebab-case}.md` 에 아래 형식으로 작성합니다.

```markdown
# 테스트 시나리오: {기능명}
작성일: {날짜}  /  리서치 기반: explore + librarian
승인 상태: ⏳ 승인 대기 중

## Happy Path
- [ ] SCENARIO: {정상 동작 설명}
  GIVEN: {초기 조건}
  WHEN:  {실행 동작}
  THEN:  {기대 결과}

## Edge Cases
- [ ] SCENARIO: {경계값 / null / 빈값 등}
  GIVEN: ...  WHEN: ...  THEN: ...

## Error Cases
- [ ] SCENARIO: {에러 처리}
  GIVEN: ...  WHEN: ...  THEN: {에러 타입·메시지}

## 모킹 전략
- {외부 의존성}: {모킹 방법 (vi.mock / vi.fn / vi.spyOn)}

## SDV 스펙 검증
- [ ] #REQ-XXX: {요구사항 항목}
```

#### 2-B. 테스트 파일 사전 생성 (Red 단계)

시나리오 항목을 기반으로 **반드시** 실패하는 테스트 파일을 먼저 작성합니다.
`{구현파일명}.test.ts` 를 소스 파일과 같은 디렉터리에 생성합니다.

```typescript
// 시나리오 파일: .opencode/test-scenarios/{파일명}.md 기반
import { describe, it, expect, vi, beforeEach } from 'vitest';

describe('{구현 대상}', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  // Happy Path
  describe('정상 동작', () => {
    it('{대상}_{조건}_{기대결과}', async () => {
      // SCENARIO #1 — happy path
      // GIVEN
      // WHEN
      // THEN
      expect(true).toBe(false); // ← 구현 전 의도적 실패
    });
  });

  // Edge Cases
  describe('경계값', () => {
    it('{대상}_{경계조건}_{기대결과}', async () => {
      // SCENARIO #N — edge case
      expect(true).toBe(false);
    });
  });

  // Error Cases
  describe('에러 처리', () => {
    it('{대상}_{에러조건}_{에러결과}', async () => {
      // SCENARIO #N — error path
      expect(true).toBe(false);
    });
  });
});
```

> **원칙**: 테스트 파일은 구현 전에 먼저 존재해야 한다. 시나리오 항목 수 = 테스트 케이스 수 (1:1).

---

### ⛔ PHASE 3: 사용자 승인 대기

시나리오 + 테스트 파일 작성 완료 후 **`ask_user_input_v0` 도구**로 반드시 승인을 요청하고 멈춥니다.
승인 전 소스 코드 생성·수정 절대 금지.

```
ask_user_input_v0(
  questions=[
    {
      question: "⛔ [승인 필요] 아래 내용을 검토 후 승인해주세요.\n\n"
               + "📋 시나리오: .opencode/test-scenarios/{파일명}.md\n"
               + "🧪 테스트 파일: {테스트파일경로}\n"
               + "🔍 Explore: {파일 수}개 파일 탐색 완료\n"
               + "🌐 Librarian: {문서 수}개 문서/패턴 조사 완료\n\n"
               + "구현을 시작할까요?",
      options: ["승인합니다 — 구현을 시작하세요", "수정이 필요합니다 — 내용을 알려주세요"],
      type: "single_select"
    }
  ]
)
```

"수정이 필요합니다"를 선택하면 수정 내용을 반영하고 승인 게이트를 재실행한다.

#### 3-B. Prisma 스키마 변경 감지 (해당 시)

PHASE 1 Explore 결과에서 `prisma/schema.prisma` 변경이 필요하다고 판단되면,
소스 구현 전에 `ask_user_input_v0`로 별도 승인을 요청한다.

```
ask_user_input_v0(
  questions=[
    {
      question: "🗄️ Prisma 스키마 변경이 감지되었습니다.\n\n"
               + "변경 내용: {변경할 모델/필드 요약}\n\n"
               + "⚠️ db push는 개발 DB에 즉시 반영됩니다. 실행할까요?",
      options: [
        "승인합니다 — prisma db push 실행",
        "건너뜁니다 — 스키마 변경 없이 진행"
      ],
      type: "single_select"
    }
  ]
)
```

승인 시:
```
!npx prisma db push
```
실패 시 즉시 사용자에게 에러를 보고하고 중단한다.

---

### ⬛ PHASE 4: 병렬 구현 위임

태스크를 성격별로 분리하여 `delegate_task()`로 병렬 실행합니다.

**구현 프롬프트 필수 6개 섹션**:

```
delegate_task(
  category='{적절한 카테고리}',
  run_in_background=True,
  prompt='''
    CONTEXT:        {작업 배경 및 목적}
    TASK:           {구체적인 구현 내용}
    CONSTRAINTS:    {금지 동작, 준수할 코드 스타일, RULES.md 참조}
    TEST_SCENARIOS: .opencode/test-scenarios/{파일명}.md 의 {관련 시나리오 항목}
    ACCEPTANCE:     {완료 기준 - 어떤 시나리오가 통과해야 하는지}
    OUTPUT:         {생성/수정할 파일 목록}

    ## TDD 실행 순서
    1. 이미 생성된 테스트 파일({테스트파일경로})을 실행하여 Red 상태 확인
       !npm test {테스트파일경로}
    2. 테스트를 통과하는 최소 구현 코드 작성 (Green)
    3. 테스트 재실행하여 Green 확인
       !npm test {테스트파일경로}

    ## JSDoc 작성 (구현과 동시에)
    신규·수정한 모든 함수/메서드/클래스/인터페이스에 RULES.md#jsdoc 기준 JSDoc을 작성한다.
    JSDoc 없이 구현 완료를 보고하는 것을 금지한다.
    - public 함수: @param, @returns, @throws 필수
    - 복잡한 로직: @example 포함
    - deprecated: @deprecated + 대체 API 명시
  '''
)
```

카테고리 선택 기준:
- `deep`               → 복잡한 분석·버그 수정·새 서비스
- `visual-engineering` → UI/UX·CSS·컴포넌트 (항상 이 카테고리 사용)
- `quick`              → 간단한 수정·설정 변경
- `ultrabrain`         → 고난도 아키텍처 설계

---

### ⬛ PHASE 5: 가비지 컬렉터 (GC)

구현 완료 후 **반드시** 실행합니다. 리팩터링 → 미사용 코드 제거 → 아키텍처 점검 순서.

#### 5-A. 리팩터링

`refactor` 스킬 또는 `deep` 카테고리로 이번 구현 파일을 대상으로 실행합니다.
RULES.md#refactoring 의 PREPARE → IDENTIFY → REFACTOR → VERIFY 절차를 따릅니다.

```
delegate_task(
  category='deep',
  prompt='''
    CONTEXT: 구현 완료 후 리팩터링 단계
    TASK:
      이번 구현에서 생성·수정한 파일({파일 목록})을 대상으로
      RULES.md#refactoring 체크리스트를 점검하고 리팩터링을 수행하라.

    체크리스트:
    - [ ] Long Method       — 50줄 초과 함수
    - [ ] Duplicated Code   — 동일·유사 로직 반복
    - [ ] Dead Code         — 미사용 변수·함수·주석
    - [ ] Magic Number      — 설명 없는 리터럴
    - [ ] Nested Conditional— 3단계 이상 중첩 조건문

    각 리팩터링 후 반드시 테스트를 재실행하여 회귀가 없음을 확인하라.
    !npm test

    CONSTRAINTS: 동작 변경 금지. 테스트 통과 유지 필수.
    OUTPUT: 리팩터링된 파일 + 처리한 코드 스멜 목록
  '''
)
```

#### 5-B. 미사용 코드·Import 제거

```
delegate_task(
  category='quick',
  prompt='''
    CONTEXT: 미사용 루틴·import 정리
    TASK:
      이번 구현 파일({파일 목록})에서 아래를 제거하라:
      1. 미사용 import 구문
      2. 미사용 변수·함수·타입
      3. 주석 처리된 Dead Code 블록
      4. console.log / 디버그용 코드

    !npm run lint -- --fix
    !npx tsc --noEmit

    CONSTRAINTS: ESLint no-unused-vars, @typescript-eslint/no-unused-imports 기준 준수.
    OUTPUT: 정리된 파일 목록
  '''
)
```

#### 5-C. 아키텍처 위반 점검

```
delegate_task(
  category='deep',
  prompt='''
    CONTEXT: AGENTS.md / RULES.md 기준 아키텍처 위반 점검
    TASK:
      이번 구현 파일({파일 목록})이 아래 규칙을 준수하는지 점검하라:

      1. DDD 레이어 경계 (RULES.md#api 기준)
         - HTTP 레이어 → Application 레이어만 호출
         - Application 레이어 → Domain 인터페이스만 참조 (구현체 직접 참조 금지)
         - Zod schema는 Domain 레이어에만 위치
      2. 임포트 경계 (모노레포)
         - 패키지 간 상대 경로 직접 임포트 금지
         - apps/* 간 직접 임포트 금지
      3. JSDoc 누락 여부 (RULES.md#jsdoc 기준)
      4. AGENTS.md의 LIVING DOCUMENT 규칙 위반 여부

      위반 항목이 있으면 즉시 수정하라.
    OUTPUT: 점검 결과 보고서 + 수정된 파일 (있을 경우)
  '''
)
```

---

### ⬛ PHASE 6: 검증

**정적 검사 → 타입 검사 → 테스트** 순서로 실행합니다.

```bash
# 1단계: 정적 검사
!npm run lint 2>&1 | tail -20

# 2단계: 타입 검사
!npx tsc --noEmit 2>&1 | tail -20

# 3단계: 시나리오 기반 테스트 전체 실행
!npm test 2>&1 | tail -40
```

테스트 결과를 `.opencode/test-scenarios/{파일명}.md` 에 반영합니다:
- 통과: `[ ]` → `[x PASS]`
- 실패: `[ ]` → `[x FAIL: {원인}]`

실패 항목이 있으면 **같은 session_id로** PHASE 4를 재실행합니다.
모든 항목 `[x PASS]` + Lint 0 + 타입 에러 0 이 될 때까지 반복합니다.

전체 통과 시:
```bash
!git add -A
!git status
```
