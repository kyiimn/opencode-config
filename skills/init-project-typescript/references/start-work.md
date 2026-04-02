---
description: "Prometheus 계획 수립 → TDD/SDV 워크플로우 - 병렬 리서치 → 시나리오 작성 → 사용자 승인 → 병렬 구현 → GC → 검증"
agent: atlas
subtask: false
---

# /start-work: $ARGUMENTS

## 사전 확인

!git status
!git log --oneline -5
!ls .opencode/test-scenarios/ 2>/dev/null && echo "기존 시나리오 있음" || echo "시나리오 없음 (새로 작성 필요)"

## 컨텍스트 로드 (필수)

모노레포 하위 패키지에서 실행 중인 경우, 현재 디렉터리의 `AGENTS.md` 외에
**루트 `RULES.md`를 반드시 명시적으로 읽는다.**
코딩 규칙은 루트 `RULES.md`에 정의되어 있으며, 서브 패키지의 `AGENTS.md`에는 포함되지 않는다.

```
@AGENTS.md
@../../RULES.md  # 모노레포 루트 (패키지 깊이에 따라 경로 조정: ../RULES.md 또는 ../../RULES.md)
```

> 루트 `RULES.md` 없이 작업을 시작하는 것을 금지한다.
> 경로를 찾을 수 없으면 `git rev-parse --show-toplevel` 로 루트를 확인한 뒤 로드한다.

---

## 실행 지침 (7단계 순서 엄수)

> ⛔ **`sisyphus-junior` 직접 호출 금지**
> `sisyphus-junior`는 카테고리 기반 병렬 위임을 내부적으로 조율하는 시스템 에이전트다.
> 구현 작업을 위임할 때 `sisyphus-junior`를 직접 호출하지 말고, `category`만 지정하면 시스템이 자동으로 처리한다.
>
> ```
> # ❌ 금지
> delegate_task(subagent_type='sisyphus-junior', ...)
>
> # ✅ 올바른 방식 — category만 지정
> delegate_task(category='deep', ...)
> delegate_task(category='quick', ...)
> ```
>
> `explore`, `librarian`, `oracle`, `metis`, `momus` 등 전문 역할 에이전트는 기존과 동일하게 직접 호출한다.

---

### ⬛ PHASE 0: Prometheus 작업 계획 수립 (PHASE 1 진입 전 필수)

`/start-work` 실행 전에 **Prometheus 에이전트가 작업 계획을 먼저 수립해야 한다.**
`.sisyphus/plans/` 에 기존 계획 파일이 있으면 해당 파일을 읽고 계획이 이미 수립됐는지 확인한다.

```
# 기존 계획 파일 확인
@.sisyphus/plans/
```

**계획이 없는 경우** — Prometheus를 직접 호출하여 계획 수립을 요청한다.

```
@prometheus $ARGUMENTS 에 대한 작업 계획서를 .sisyphus/plans/{task-kebab-case}.md 에 작성해줘.

계획서에는 아래 항목을 포함할 것:

## 작업 목표
{요구사항을 측정 가능한 완료 조건으로 명확히 기술}

## 작업 범위
- 신규 생성할 파일/모듈
- 수정이 필요한 기존 파일/모듈
- 변경 범위 밖 (영향받지 않아야 하는 것)

## 구현 접근법
{어떤 방식으로 구현할지 단계별 기술}

## 리스크 및 주의사항
{예상되는 문제점, 의존성 이슈, 기술적 제약}

## 완료 기준
- [ ] {검증 가능한 완료 조건 1}
- [ ] {검증 가능한 완료 조건 2}

원칙:
- 모든 완료 조건은 측정 가능해야 한다. "잘 동작한다", "빠르다" 같은 표현은 금지.
- 코드를 직접 생성하지 않는다. 계획 문서만 작성한다.
```

**계획 수립 후 사용자 승인** — `ask_user_input_v0`으로 계획 확인을 요청한다.

```
ask_user_input_v0(
  questions=[
    {
      question: "Prometheus가 작업 계획을 수립했습니다. 계획 파일: .sisyphus/plans/{파일명}.md. 이 계획으로 구현을 진행할까요?",
      options: ["승인합니다 — PHASE 1 리서치를 시작하세요", "수정이 필요합니다 — 내용을 알려주세요"],
      type: "single_select"
    }
  ]
)
```

"수정이 필요합니다"를 선택하면 `@prometheus` 를 다시 호출하여 계획을 수정하고 승인 게이트를 재실행한다.
**계획 승인 없이 PHASE 1 이후로 진행하는 것을 절대 금지한다.**

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
      question: "시나리오 파일 .opencode/test-scenarios/{파일명}.md 과 테스트 파일 {테스트파일경로} 작성이 완료됐습니다. Explore {파일 수}개 파일, Librarian {문서 수}개 문서 탐색 완료. 구현을 시작할까요?",
      options: ["승인합니다", "수정이 필요합니다"],
      type: "single_select"
    }
  ]
)
```

"수정이 필요합니다"를 선택하면 수정 내용을 반영하고 승인 게이트를 재실행한다.

---

### ⬛ PHASE 4: 병렬 구현 위임

태스크를 성격별로 분리하여 `delegate_task()`로 병렬 실행합니다.

**구현 프롬프트 필수 6개 섹션**:

> **패키지 매니저**: `pnpm-lock.yaml` 존재 시 `pnpm` 사용, 없으면 `npm` 사용.
> 모노레포 환경에서는 항상 `pnpm` 을 사용한다.

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
       !pnpm test {테스트파일경로}   # 모노레포 / pnpm 프로젝트
       !npm test {테스트파일경로}    # npm 단일 프로젝트
    2. 테스트를 통과하는 최소 구현 코드 작성 (Green)
    3. 테스트 재실행하여 Green 확인

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

구현 완료 후 **반드시** 실행합니다.
**5-A → 5-B → 5-C 순서로 순차 실행합니다. 병렬 실행을 금지합니다.**
각 단계는 이전 단계가 완료된 후에만 시작합니다.

#### 5-A. 리팩터링 (순서 1)

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
    모노레포: !pnpm test / 단일: !npm test

    CONSTRAINTS: 동작 변경 금지. 테스트 통과 유지 필수.
    OUTPUT: 리팩터링된 파일 + 처리한 코드 스멜 목록
  '''
)
```

#### 5-B. 미사용 코드·Import 제거 (순서 2)

5-A 완료 후 실행합니다.

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

    모노레포: !pnpm run lint -- --fix / 단일: !npm run lint -- --fix
    !npx tsc --noEmit

    CONSTRAINTS: ESLint no-unused-vars, @typescript-eslint/no-unused-imports 기준 준수.
    OUTPUT: 정리된 파일 목록
  '''
)
```

#### 5-C. 아키텍처 위반 점검 (순서 3)

5-B 완료 후 실행합니다. **현재 패키지 타입에 해당하는 항목만 점검합니다.**

```
delegate_task(
  category='deep',
  prompt='''
    CONTEXT: AGENTS.md / RULES.md 기준 아키텍처 위반 점검
    TASK:
      이번 구현 파일({파일 목록})을 패키지 타입에 따라 아래 항목으로 점검하라.
      해당하지 않는 스택의 항목은 건너뛴다.

    ## 공통 (모든 스택)
    - JSDoc 누락 여부 (RULES.md#jsdoc 기준)
    - 모노레포 임포트 경계: 패키지 간 상대 경로 직접 임포트 금지, apps/* 간 직접 임포트 금지
    - AGENTS.md LIVING DOCUMENT 규칙 위반 여부

    ## API 서버 (Express.js + DDD) — RULES.md#api
    - HTTP 레이어 → Application 레이어만 호출 (Domain 직접 호출 금지)
    - Application 레이어 → Domain 인터페이스만 참조 (구현체 직접 참조 금지)
    - Zod schema는 Domain 레이어에만 위치 (Controller 직접 정의 금지)
    - Controller 메서드는 반드시 arrow function
    - Entity/DTO 분리 준수

    ## 프론트엔드 (React 19+) — RULES.md#frontend
    - useEffect 데이터 페칭 금지 (React Query / Server Action 사용)
    - any 타입 props 금지
    - "use client" 최소화 — Server Component 우선
    - 단일 파일 내 주 컴포넌트 export 하나만

    ## CLI 앱 — RULES.md#cli
    - 커맨드마다 단일 책임 (파싱 → 비즈니스 로직 위임 → 출력)
    - exit code 준수: 성공 0 / 사용자 오류 1 / 시스템 오류 2
    - 오류는 stderr 출력, 정상 결과는 stdout 출력

    ## 공유 라이브러리 — RULES.md#shared
    - src/index.ts 가 유일한 공개 진입점
    - apps/* 패키지 임포트 금지
    - Breaking change 시 semver major 버전 업 여부 확인

    위반 항목이 있으면 즉시 수정하라.
    OUTPUT: 점검 결과 보고서 + 수정된 파일 (있을 경우)
  '''
)
```

---

### ⬛ PHASE 6: 검증

**정적 검사 → 타입 검사 → Prisma db push(해당 시) → 테스트** 순서로 실행합니다.
모노레포(`pnpm-lock.yaml` 존재) 환경에서는 `pnpm` 명령을, 그 외에는 `npm` 명령을 사용합니다.

```bash
# 1단계: 정적 검사
!pnpm run lint 2>&1 | tail -20    # 모노레포 / pnpm
!npm run lint 2>&1 | tail -20     # npm 단일 프로젝트

# 2단계: 타입 검사
!npx tsc --noEmit 2>&1 | tail -20
```

#### 6-A. Prisma 스키마 변경 감지 (해당 시) — 테스트 직전 필수

PHASE 4 구현 중 `prisma/schema.prisma` 가 신규 생성되거나 수정됐다면, 테스트 실행 전에 반드시
`ask_user_input_v0`로 db push 승인을 요청한다. `.env` 파일은 구현 단계에서 생성되므로
이 시점에 db push가 가능하다. 테스트보다 먼저 실행하지 않으면 DB 연결 오류로 테스트가 실패한다.

```
ask_user_input_v0(
  questions=[
    {
      question: "prisma/schema.prisma 가 변경됐습니다. 변경 내용: {변경된 모델/필드 요약}. db push를 실행하면 개발 DB에 즉시 반영됩니다. 테스트 전에 실행할까요?",
      options: ["승인합니다 — prisma db push 실행", "건너뜁니다 — 스키마 변경 없이 진행"],
      type: "single_select"
    }
  ]
)
```

승인 시:
```bash
!npx prisma db push
```
실패 시 즉시 사용자에게 에러를 보고하고 중단한다.

#### 6-B. 시나리오 기반 테스트

```bash
# 3단계: 시나리오 기반 테스트 전체 실행
!pnpm test 2>&1 | tail -40    # 모노레포 / pnpm
!npm test 2>&1 | tail -40     # npm 단일 프로젝트
```

테스트 결과를 `.opencode/test-scenarios/{파일명}.md` 에 반영합니다:
- 통과: `[ ]` → `[x PASS]`
- 실패: `[ ]` → `[x FAIL: {원인}]`

실패 항목이 있으면 PHASE 4를 재실행합니다.
**새 에이전트를 만들지 말고 반드시 동일 session_id로 재위임합니다.**

```
# PHASE 4 재위임 시 session_id 재사용 방법
delegate_task(
  category='{동일 카테고리}',
  session_id='{PHASE 4에서 사용한 session_id}',  # 기존 session 이어받기
  prompt='''
    이전 구현에서 실패한 시나리오를 수정하라.

    실패 항목:
    {.opencode/test-scenarios/{파일명}.md 의 [x FAIL] 항목 목록}

    실패 원인 분석:
    {에러 메시지 및 원인}
  '''
)
```

모든 항목 `[x PASS]` + Lint 0 + 타입 에러 0 이 될 때까지 반복합니다.

전체 통과 시:
```bash
!git add -A
!git status
```
