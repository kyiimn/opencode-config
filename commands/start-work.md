---
description: "TDD/SDV 워크플로우 - 병렬 탐색 → 시나리오 작성 → 사용자 승인 → 병렬 구현 → 검증"
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

## 실행 지침 (5단계 순서 엄수)

### ⬛ PHASE 1: 병렬 리서치

아래 에이전트들을 **동시에** 백그라운드로 실행하세요.
모든 결과가 돌아올 때까지 다음 단계로 넘어가지 마세요.

**코드베이스 탐색** — explore 에이전트 (background)
```
delegate_task(
  subagent_type='explore',
  run_in_background=True,
  prompt='''
    CONTEXT: $ARGUMENTS 구현을 위한 코드베이스 탐색
    TASK:
      1. 작업 범위와 관련된 소스 파일 목록 (전체 경로)
      2. 기존 테스트 파일 위치 및 테스트 프레임워크 (Jest/Vitest/pytest 등)
      3. 재사용 가능한 유사 구현 패턴
      4. 수정 시 영향받는 의존 모듈
      5. .opencode/test-scenarios/ 내 기존 시나리오 파일
    OUTPUT: 구조화된 탐색 보고서
  '''
)
```

**라이브러리/문서 조사** — librarian 에이전트 (background)
```
delegate_task(
  subagent_type='librarian',
  run_in_background=True,
  prompt='''
    CONTEXT: $ARGUMENTS 구현에 필요한 라이브러리/API 조사
    TASK:
      1. 관련 라이브러리 공식 API 및 버전
      2. 테스트 유틸리티 / 모킹 라이브러리 사용법
      3. 알려진 버그나 제한사항
    OUTPUT: 라이브러리 조사 보고서
  '''
)
```

**외부 참고 자료** — 필요한 경우에만 실행
- websearch MCP: 최신 베스트 프랙티스, 라이브러리 이슈 검색
- grep_app MCP: 유사 오픈소스 구현 패턴 검색
- context7 MCP: 공식 문서 최신 버전 확인

---

### ⬛ PHASE 2: TDD/SDV 테스트 시나리오 작성

리서치 결과를 반영하여 `.opencode/test-scenarios/` 에 시나리오 파일을 생성하세요.
아래 형식을 반드시 따르세요:

```markdown
# 테스트 시나리오: {기능명}
작성일: {날짜}  /  리서치 기반: explore + librarian

## Happy Path
- [ ] SCENARIO: {정상 동작}
  GIVEN: {초기 조건}
  WHEN:  {실행 동작}
  THEN:  {기대 결과}

## Edge Cases
- [ ] SCENARIO: {경계값 / null / 빈값}
  GIVEN / WHEN / THEN: ...

## Error Cases
- [ ] SCENARIO: {에러 처리}
  GIVEN / WHEN / THEN: ...

## 모킹 전략
- {외부 의존성}: {모킹 방법}

## SDV 스펙 검증
- [ ] #REQ-XXX: {요구사항 항목}
```

---

### ⛔ PHASE 3: 사용자 승인 대기 (구현 시작 전 차단)

시나리오 작성 완료 후 반드시 출력하고 대기하세요:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⛔ [승인 필요] 구현 시작 전 테스트 시나리오를 검토해주세요.

   작성된 시나리오: .opencode/test-scenarios/{파일명}.md

   ✅ 승인:      '승인' 또는 'approve'
   ✏️  수정 요청: 변경할 내용을 입력해주세요
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**승인 전 절대 금지:**
- 소스 코드 파일 생성/수정
- 테스트 파일 생성
- 패키지 설치

---

### ⬛ PHASE 4: 병렬 구현 위임 (승인 후)

태스크를 성격에 따라 분리하여 delegate_task()로 병렬 실행.

각 호출 시 아래 6개 섹션을 프롬프트에 포함하세요:
```
delegate_task(
  category='{적절한 카테고리}',
  run_in_background=True,  # 독립적 태스크는 병렬
  prompt='''
    CONTEXT:        {작업 배경 및 목적}
    TASK:           {구체적인 구현 내용}
    CONSTRAINTS:    {금지 동작, 준수할 코드 스타일}
    TEST_SCENARIOS: .opencode/test-scenarios/{파일명}.md 의 {관련 시나리오}
    ACCEPTANCE:     {완료 기준 - 어떤 시나리오가 통과해야 하는지}
    OUTPUT:         {생성/수정할 파일 목록}
  '''
)
```

카테고리 선택 기준:
- `deep`             → 복잡한 분석, 버그 수정, 리팩터링
- `business-logic`   → 도메인 로직, 데이터 처리
- `visual-engineering` → UI/UX, CSS, 컴포넌트 (항상 이 카테고리 사용)
- `quick`            → 간단한 수정, 설정 변경
- `ultrabrain`       → 고난도 아키텍처 설계

---

### ⬛ PHASE 5: 검증

구현 완료 후 순서대로 실행하고 결과를 보고하세요:

**기본 정적 검사**
!npm run lint 2>&1 | tail -20
!npx tsc --noEmit 2>&1 | tail -20

**시나리오 기반 테스트**
!npm test 2>&1 | tail -40

테스트 실패 시:
- 시나리오 파일에서 해당 항목을 `[ ]` → `[x FAIL: 원인]` 으로 표시
- 원인을 분석하여 해당 카테고리로 재위임
- 모든 항목이 `[x PASS]` 될 때까지 반복

전체 통과 시:
!git add -A
!git status
