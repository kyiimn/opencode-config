---
name: gemini-test-delegate
description: |
  OpenCode로 구현 작업이 완료된 후, 해당 구현에 대한 테스트 코드 작성을 Gemini CLI에 위임하여 교차 검증을 수행하는 스킬.
  다음 상황에서 반드시 이 스킬을 사용하세요:
  - OpenCode 위임(delegation) 작업이 완료되고 테스트 코드를 작성해야 할 때
  - "gemini에 테스트 위임", "교차 검증", "gemini로 테스트 작성" 등을 언급할 때
  - oh-my-opencode 워크플로우에서 구현 완료 후 검증 단계가 필요할 때
  - 구현자와 다른 AI가 테스트를 작성하게 하여 편향 없는 검증이 필요할 때
---

# Gemini CLI 테스트 위임 스킬

OpenCode에서 구현된 코드에 대해 Gemini CLI를 통해 독립적으로 테스트 코드를 작성하고 교차 검증하는 워크플로우.

---

## 개요

```
[OpenCode 구현 완료]
        ↓
[이 스킬 실행]
        ↓
[Gemini CLI 위임 프롬프트 생성]
        ↓
[gemini -p "..." 명령어 출력]
        ↓
[테스트 파일 생성 및 검증]
```

---

## Step 1: 구현 컨텍스트 수집

위임 블록에서 다음 정보를 추출하세요:

| 필드 | 위임 블록의 대응 항목 |
|------|----------------------|
| `target_files` | `작업 대상 파일` 목록 |
| `role` | `구현 명세 > 역할` |
| `inputs` | `구현 명세 > 입력` |
| `outputs` | `구현 명세 > 출력` |
| `constraints` | `구현 명세 > 제약` |
| `rules_ref` | `참조할 규칙` |

추출이 불명확한 경우 사용자에게 확인 후 진행합니다.

---

## Step 2: 구현 파일 읽기

```bash
# 구현된 소스 파일의 실제 내용을 읽어 시그니처·타입·export를 파악
cat {구현된_파일_경로}
```

파일 내용을 바탕으로:
- export된 함수/클래스/타입 목록 확인
- 실제 파라미터명과 타입 확인 (위임 명세와 다를 수 있음)
- 예외를 throw하는 케이스 식별

---

## Step 3: Gemini CLI 위임 프롬프트 생성

아래 템플릿을 채워 `gemini -p "..."` 명령어를 생성합니다.

### 프롬프트 템플릿

```
다음 TypeScript 구현에 대한 Vitest 테스트 코드를 작성해주세요.

## 구현 정보

**파일**: {구현_파일_경로}
**역할**: {역할_설명}
**입력**: {파라미터: 타입}
**출력**: {반환_타입}
**제약**: {레이어_규칙_및_금지_패턴}

## 구현 코드

\`\`\`typescript
{실제_구현_코드}
\`\`\`

## 테스트 작성 규칙

### 필수 준수사항
- **Vitest 사용** — Jest 절대 금지 (`import { describe, it, expect, vi } from 'vitest'`)
- **테스트 파일 위치**: `{구현_파일_경로_기반}.test.ts` (소스와 같은 디렉터리)
- **커버리지 목표**: 핵심 비즈니스 로직 80% 이상

### 테스트 구조
\`\`\`typescript
describe('{대상_모듈명}', () => {
  describe('{함수명}', () => {
    it('{대상}_{조건}_{기대결과}', () => {
      // given
      // when
      // then
    })
  })
})
\`\`\`

### 테스트명 패턴
- 형식: `{대상}_{조건}_{기대결과}`
- 예: `getUser_존재하지않는id_NotFoundError반환`

### 커버해야 할 케이스
1. **Happy path**: 정상 입력에 대한 기대 출력 (함수·메서드마다 최소 1개)
2. **Edge case**: 경계값, 빈 배열/문자열, null/undefined
3. **Error path**: 잘못된 입력, 예외 발생 케이스

### 모킹 규칙
- 외부 의존성은 `vi.mock()` 사용
- 함수 모킹: `vi.fn()`, `vi.spyOn()`
- 각 테스트 후 `vi.clearAllMocks()` 또는 `afterEach` 정리

### 금지 패턴
{제약에서_추출한_금지_패턴}

## 출력 형식
테스트 파일 전체 코드만 출력하세요. 설명은 코드 주석으로만 작성합니다.
파일 경로를 첫 줄 주석으로 명시하세요: `// {테스트_파일_경로}`
```

---

## Step 4: 명령어 실행 방법 안내

생성된 프롬프트를 바탕으로 다음 형식으로 실행 명령어를 제시합니다:

### 방법 A: 직접 파이프 (구현 파일을 컨텍스트로 전달)

```bash
cat {구현_파일_경로} | gemini -p "{생성된_프롬프트}"
```

### 방법 B: 파일 참조 플래그 사용

```bash
gemini -p "{생성된_프롬프트}" --file {구현_파일_경로}
```

### 방법 C: 프롬프트 파일로 저장 후 실행 (긴 프롬프트)

```bash
# 프롬프트를 임시 파일로 저장
cat > /tmp/test-delegate-prompt.txt << 'EOF'
{생성된_프롬프트}
EOF

gemini -p "$(cat /tmp/test-delegate-prompt.txt)"
```

---

## Step 5: 결과 검증 체크리스트

Gemini가 생성한 테스트 코드를 검토할 때 확인사항:

```
□ import가 'vitest'에서 올바르게 됨 (jest import 없음)
□ 테스트 파일 경로가 소스와 같은 디렉터리
□ 테스트명이 {대상}_{조건}_{기대결과} 패턴 준수
□ describe > it 중첩 구조 사용
□ happy path 최소 1개 포함
□ edge case 포함
□ error path 포함
□ 외부 의존성 vi.mock() 처리
□ 금지 패턴 미사용
□ TypeScript 타입 오류 없음
```

테스트 실행으로 최종 검증:

```bash
npx vitest run {테스트_파일_경로}
# 또는 커버리지 포함
npx vitest run --coverage {테스트_파일_경로}
```

---

## 전체 위임 블록 → 명령어 변환 예시

> 참조: `references/example.md`

---

## 주의사항

- Gemini CLI가 설치되지 않은 경우: `npm install -g @google/gemini-cli` 또는 `pip install google-generativeai`
- 구현 코드가 길 경우 (`--file` 플래그 사용 권장)
- 모노레포 환경에서는 `vitest.config.ts` 경로를 함께 전달하면 정확도 향상
- 테스트 생성 후 **반드시 `npx vitest run`으로 실제 실행**하여 통과 여부 확인
