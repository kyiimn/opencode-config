# 템플릿 — AGENTS.md 섹션

모든 `[A]`, `[A:ROOT]`, `[A:PKG]` 섹션 모음.
조합 규칙은 `template.md`를 참조한다.

---

## [A:ROOT] HEADER

```markdown
# {PROJECT_NAME} — AGENTS.md

**Stack:** TypeScript (strict) · {STACK_SUMMARY}
**Package Manager:** {npm | pnpm}
**Rules:** 상세 코딩 규칙 → [`RULES.md`](./RULES.md)
```

---

## [A:PKG] HEADER

```markdown
# {PACKAGE_PATH} — AGENTS.md

> 루트 [`/AGENTS.md`](/AGENTS.md) 전역 규칙 + [`/RULES.md`](/RULES.md) 상속.
> 이 파일은 `{PACKAGE_PATH}` 패키지에 특화된 구조 정보만 기술한다.

**패키지 역할:** {담당 책임 한 줄}
**패키지 타입:** {API 서버 앱 | Frontend 앱 | CLI 앱 | 공유 라이브러리}
```

---

## [A:ROOT] OVERVIEW — 설명이 있을 때

```markdown
## OVERVIEW

{프로젝트 설명 2~4문장}
```

---

## [A:ROOT] STRUCTURE (싱글 프로젝트)

```markdown
## PROJECT STRUCTURE

```
{project-name}/
├── src/
│   ├── index.ts
│   ├── {domain}/
│   └── types/
├── tests/
├── AGENTS.md
├── RULES.md
├── package.json
└── tsconfig.json
```
```

---

## [A:ROOT] STRUCTURE (모노레포 pnpm)

```markdown
## PROJECT STRUCTURE

pnpm workspaces 기반 모노레포.

```
{project-name}/
├── apps/               # 배포 단위 애플리케이션 (패키지별 AGENTS.md 존재)
├── packages/
│   └── core/           # 모노레포 공유 설정·라이브러리
│       ├── tsconfig/
│       │   ├── base.json        # 모든 패키지가 extends하는 기본 tsconfig
│       │   ├── app.json         # 앱(Node/Bun) 전용 tsconfig
│       │   └── react-lib.json   # React 라이브러리 전용 tsconfig
│       ├── eslint/
│       │   ├── index.js         # 공통 ESLint flat config (base)
│       │   ├── react.js         # React 패키지용 ESLint 확장
│       │   └── node.js          # Node/API 패키지용 ESLint 확장
│       └── src/
│           └── index.ts         # 공유 타입·유틸리티 진입점
├── AGENTS.md
├── RULES.md
├── pnpm-workspace.yaml
├── package.json
└── tsconfig.json       # 루트 tsconfig (packages/core/tsconfig/base.json extends)
```

- `apps/` — 배포 단위 애플리케이션. 각 앱은 `packages/core/tsconfig`와 `packages/core/eslint`를 참조한다
- `packages/core/` — 모노레포 전체가 공유하는 tsconfig·ESLint 설정 및 공유 라이브러리

**임포트 규칙**: 패키지 이름으로 임포트. 상대 경로로 경계를 넘지 않는다.

**설정 참조 패턴**:
```jsonc
// 각 패키지의 tsconfig.json
{ "extends": "@{scope}/core/tsconfig/app.json" }

// 각 패키지의 eslint.config.js
import baseConfig from '@{scope}/core/eslint/node.js';
export default [...baseConfig];
```
```

---

## [A:ROOT] ARCHITECTURE — 핵심 파일 경로 (있을 때)

```markdown
## ARCHITECTURE

에이전트는 아래 파일을 작업 전에 먼저 확인한다:

| 역할 | 경로 |
|------|------|
| 서버 진입점 | `{예: apps/api/src/index.ts}` |
| 라우터 정의 | `{예: apps/api/src/routes/index.ts}` |
| API 클라이언트 | `{예: apps/web/src/lib/api.ts}` |
| 상태 관리 | `{예: apps/web/src/stores/}` |
| 공유 타입 | `{예: packages/core/src/index.ts}` |
| 환경변수 설정 | `{예: apps/api/src/config.ts}` |
```

---

## [A:ROOT] CODE EXAMPLES — Good/Bad 파일이 있을 때

```markdown
## CODE EXAMPLES

### 참고할 파일 (Good)
- {예: `src/modules/user/application/user.service.ts`} — Service 레이어 패턴
- {예: `src/components/UserCard.tsx`} — 컴포넌트 작성 패턴
- {예: `src/hooks/useData.ts`} — 데이터 페칭 훅 패턴

### 참고 금지 파일 (Legacy / Bad)
- {예: `src/legacy/OldAdmin.tsx`} — 클래스 기반, 현재 패턴과 다름
- {예: `src/utils/deprecated.ts`} — 삭제 예정, 임포트 금지
```

---

## [A:ROOT] COMMANDS

```markdown
## COMMANDS

| 목적 | 명령어 | 실행 위치 |
|------|--------|-----------|
| 타입 검사 | `pnpm tsc --noEmit` | 패키지 디렉터리 |
| 테스트 (단일) | `pnpm vitest run {파일경로}` | 패키지 디렉터리 |
| 테스트 (전체) | `pnpm vitest run` | 루트 |
| 린트 | `pnpm eslint --fix {파일경로}` | 루트 |
| 빌드 | `pnpm build` | **명시적 요청 시에만** |

> 타입 검사·테스트는 구현 완료 후 자율 실행. 빌드·배포는 사용자 승인 후 실행.
```

---

## [A:ROOT] SAFETY & PERMISSIONS

```markdown
## SAFETY & PERMISSIONS

### 자율 실행 가능
- 파일 읽기, 목록 조회
- `tsc --noEmit`, `eslint`, `prettier`
- 단일 테스트 파일 실행

### 반드시 사용자 승인 후 실행
- 패키지 설치·삭제 (`pnpm add`, `pnpm remove`)
- `git push`, 브랜치 생성, PR 생성
- 파일·디렉터리 삭제
- 전체 빌드·배포
- `.env` 파일 수정
```

---

## [A] AGENT WORKFLOW

```markdown
## AGENT WORKFLOW

모든 구현은 **계획 → 구현 → 테스트 코드 작성 → 검증 → 리팩터링** 루프를 완료해야 끝난다.

```
┌─────────────────────────────────────────────────────────┐
│  1. 계획: 구현 명세 + 테스트 케이스 시나리오 정의          │
│           + 사용자 승인                                  │
└────────────────────────┬────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│  2. 구현: 코드 + JSDoc 동시 작성                         │
│     ↳ 상세: RULES.md#jsdoc                             │
└────────────────────────┬────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│  3. 테스트 코드 작성: 승인된 시나리오 → .test.ts 변환     │
│     ↳ 상세: RULES.md#testing                           │
└────────────────────────┬────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│  4. 검증: tsc → eslint → vitest (자동 피드백 루프)       │
└──────────┬──────────────────────┬───────────────────────┘
           │ PASS                 │ FAIL
           ▼                      ▼
┌──────────────────────┐    실패 원인 분석
│  5. 리팩터링         │    → 1단계로 복귀
│  ↳ RULES.md#refactoring │
└──────────┬───────────┘
           │ 완료
           ▼
        작업 종료
```

### 1단계: 계획

구현 명세와 테스트 케이스 시나리오를 함께 작성하고 사용자 승인을 받는다.
승인 전에 구현을 시작하는 것을 금지한다.

```
### [추가 / 수정] {파일 경로}
- 역할: {한 문장}
- 입력: {파라미터명: 타입}
- 출력: {반환 타입 또는 사이드이펙트}
- 제약: {레이어 규칙·금지 패턴 — 해당 시만 기재}
- 변경 이유: {수정 시만 기재}

#### 테스트 케이스 시나리오
| # | 경로 | 입력 조건 | 기대 결과 |
|---|------|-----------|-----------|
| 1 | happy path | {정상 입력} | {반환값 또는 상태} |
| 2 | edge case  | {경계 입력} | {반환값 또는 상태} |
| 3 | error path | {잘못된 입력} | {예외 타입·메시지} |
```

> **그린필드 초기화 규칙**
> `AGENTS.md`(또는 `RULES.md`)만 존재하고 `src/` 또는 `apps/` 디렉터리가 없는 상태에서
> 프로젝트 구조를 초기화한 경우, 초기화 완료 즉시 아래 검증 파이프라인을 **자동 실행**한다.
> 검증 없이 초기화 완료를 보고하는 것을 금지한다.
>
> ```bash
> pnpm tsc --noEmit              # 타입 설정 오류 확인
> pnpm eslint --fix .            # 전체 파일 lint 자동 수정 후 잔여 에러 확인
> pnpm vitest run                # 초기 테스트(있을 경우) 통과 확인
> ```
>
> 검증 파이프라인 통과 후 완료를 보고한다.
> 명령어가 존재하지 않거나 설정 파일이 없어 실행 불가한 경우 즉시 사용자에게 보고한다.

### 2단계: 구현

구현 코드와 JSDoc을 동시에 작성한다. 어느 한쪽만 작성하는 것을 금지한다.
상세 JSDoc 규칙: `RULES.md#jsdoc`

### 3단계: 테스트 코드 작성 (승인된 시나리오 기반)

1단계에서 승인된 테스트 케이스 시나리오를 그대로 `{구현파일}.test.ts`로 변환한다.

**원칙**
- 시나리오에 없는 케이스를 임의로 추가하는 것을 금지한다
- 시나리오와 다른 기대값으로 작성하는 것을 금지한다
- 구현 코드를 보고 테스트를 역산하는 것을 금지한다 — 시나리오가 기준이다

**작성 방식 분기** (RULES.md#testing 설정 기준)

```
직접 작성 모드  → 에이전트가 직접 {구현파일}.test.ts 작성 후 4단계 진입
Gemini 위임 모드 → gemini-test-delegate 스킬 로드
                   → 승인된 시나리오 목록과 함께 위임
                   → 테스트 파일 확보 후 4단계 진입
```

### 4단계: 검증 (자동 피드백 루프)

**검증 파이프라인 (순서 엄수)**
```bash
pnpm tsc --noEmit              # Step 1: 타입 에러 먼저 제거
pnpm eslint --fix {파일경로}   # Step 2: lint 자동 수정 후 잔여 에러 확인
pnpm vitest run {파일}         # Step 3: 단일 파일 테스트
pnpm vitest run                # Step 4: 전체 회귀 테스트
```

vitest 실패는 **구현이 승인된 명세를 벗어난 것**으로 판단한다.
단순 코드 수정 후 재실행 금지 — 반드시 아래 순서를 따른다:

```
명령어 실행
    │
    ▼
exit code ≠ 0 또는 stderr 존재?
    │ YES
    ▼
에러 출력 전문을 컨텍스트에 캡처
    │
    ▼
에러 분류
    ├─ TypeScript 타입 에러  → 파일명:라인:열 + 메시지 파싱
    ├─ ESLint 에러           → rule ID + 파일명:라인 파싱
    ├─ Vitest 실패           → describe/it 경로 + expect 실패값 파싱
    └─ 런타임/툴 에러        → stderr 전문 + exit code 기록
    │
    ▼
원인 분석 → 어느 레이어·파일·가정이 잘못됐는지 특정
    │
    ▼
1단계(계획)로 복귀: 명세·시나리오 수정 후 사용자 재승인
```

에러 목록 전체를 수집하여 패턴(동일 파일 반복, 동일 타입 반복)을 파악한 뒤 근본 원인을 하나로 특정한다.
툴 장애·환경 문제(명령어 자체 실행 불가, 타임아웃)는 즉시 중단 후 사용자 보고.

루프 3회 초과 시 자동 중단 (OBSERVABILITY 참조).

### 5단계: 리팩터링

상세 절차: `RULES.md#refactoring`

이번 구현에서 작성·수정한 파일을 대상으로 코드 스멜을 점검하고
RULES.md의 PREPARE → IDENTIFY → REFACTOR → VERIFY 절차를 따른다.
완료 후 `tsc --noEmit` + `vitest run` 재검증 필수.
```

---

## [A] OBSERVABILITY

```markdown
## OBSERVABILITY

### 세션 트레이스

작업 시작 시:
```
## TRACE [{ISO 타임스탬프}] {작업 제목} — Loop 0
```

루프마다:
```
## TRACE [{타임스탬프}] Loop {N}
- 단계: {계획|구현|테스트 코드 작성|검증}
- 상태: {진행중|PASS|FAIL}
- 실패 시: 에러유형 / 에러위치 / 핵심메시지 / 근본원인
  (근본원인: 프롬프트오해|명세오류|구현버그|툴장애)
```

### 루프 상한: 3회 초과 시 자동 중단

```
## TRACE [중단] Loop {N} — 루프 상한 초과
Loop 1~N 실패 요약 / 패턴 분석 / 추정 원인 / 권장 조치
```

### 툴 에러 분류

| 유형 | 처리 |
|------|------|
| tsc / vitest 실패 | 자동 피드백 루프 재진입 |
| 툴 장애·환경 문제 | 즉시 중단 후 사용자 보고 |
```

---

## [A] LIVING DOCUMENT

```markdown
## LIVING DOCUMENT

> "에이전트가 실수할 때마다 AGENTS.md에 규칙을 추가한다." — Mitchell Hashimoto

### 규칙 추가 트리거
- 같은 실수 2회 이상 반복
- 검증 루프 2회 이상 실패 패턴
- 명시되지 않은 판단으로 예상 밖 결과
- 사용자가 방향 수정 필요

### 추가 형식
```
## [날짜] {한 줄 요약}
- 배경: {어떤 상황에서}
- 규칙: {에이전트가 반드시 따를 행동 — 명령형}
```
해당 섹션에 직접 추가하거나 `## LEARNED RULES`에 누적한다.

### 원칙
- 규칙은 구체적으로: "주의한다" (X) → "~할 때는 반드시 ~한다" (O)
- 규칙은 에이전트가 읽는다는 것을 염두에 두고 명령형으로 작성
- 낡은 규칙은 정기적으로 검토하여 불필요한 것은 제거한다
```

---

## [A:ROOT] GIT

```markdown
## GIT CONVENTIONS

- 브랜치: `feature/{task}`, `fix/{issue}`, `chore/{task}`
- 커밋: Conventional Commits (`feat:` `fix:` `chore:` `docs:` `refactor:` `test:`)
- PR 전 `tsc --noEmit` 및 lint 통과 필수
```

---

## [A:ROOT] FILE REFERENCES

```markdown
## FILE REFERENCES

아래 파일을 **현재 태스크와 관련된 경우에만** Read 툴로 로드한다.
`RULES.md` 전체를 한 번에 로드하지 않는다. 필요한 섹션 앵커만 지정한다.

| 언제 읽는가 | 파일 및 섹션 |
|-------------|-------------|
| 코드 작성·수정 시 | `RULES.md#code-standards` |
| JSDoc 작성 시 | `RULES.md#jsdoc` |
| API 개발 시 | `RULES.md#api` |
| Frontend 개발 시 | `RULES.md#frontend` |
| CLI 개발 시 | `RULES.md#cli` |
| 공유 라이브러리 개발 시 | `RULES.md#shared` |
| 테스트 코드 작성 시 | `RULES.md#testing` |
| 리팩터링 시 | `RULES.md#refactoring` |
| DB 스키마 확인 | `prisma/schema.prisma` |
| 환경변수 확인 | `.env.example` |
```

---

## [A:ROOT] ORCHESTRATION

```markdown
## AGENT ORCHESTRATION (oh-my-opencode / Sisyphus)

### 에이전트 역할
- **Sisyphus**: 태스크 분해·위임. 직접 코드 작성 금지
- **Hephaestus**: 코드 구현 (파일 수정·생성)
- **Oracle/Prometheus**: 계획 수립·아키텍처 검토
- **Librarian/Explore**: 코드베이스 탐색·문서 조회

### Sisyphus → Hephaestus 핸드오프 규격

자연어 단독 지시 금지. 반드시 아래 구조로 전달한다.
테스트 케이스 시나리오는 위임 블록에 반드시 포함해야 한다 — 누락 시 착수 금지.

```
## 위임: {작업 제목}

- **작업 대상 파일**:
  - `{경로}` — 신규 생성 | 기존 수정

- **구현 명세**:
  - 역할: {책임}
  - 입력: {파라미터명: 타입}
  - 출력: {반환 타입}
  - 제약: {레이어 규칙, 금지 패턴}

- **테스트 케이스 시나리오**:
  | # | 경로 | 입력 조건 | 기대 결과 |
  |---|------|-----------|-----------|
  | 1 | happy path | {정상 입력} | {반환값 또는 상태} |
  | 2 | edge case  | {경계 입력} | {반환값 또는 상태} |
  | 3 | error path | {잘못된 입력} | {예외 타입·메시지} |

- **참조할 규칙**:
  - RULES.md#{관련 섹션}
```

### AGENTS.md 계층 구조 (모노레포)

새 패키지에 AGENTS.md가 추가될 때마다 이 목록을 업데이트한다.

- `/AGENTS.md` — 프로젝트 전체 규칙 (이 파일)
<!-- 패키지 AGENTS.md 추가 시 아래에 항목을 추가하세요 -->
<!-- 예: - `/apps/api/AGENTS.md` — REST API 서버 (DDD 아키텍처) -->

### opencode.json (선택)

```json
{
  "instructions": [
    "AGENTS.md",
    "RULES.md",
    "apps/*/AGENTS.md",
    "packages/*/AGENTS.md"
  ]
}
```
```

---

## [A:ROOT] SKILL POINTERS

```markdown
## SKILL POINTERS

특정 작업을 시작하기 전에 아래 스킬을 반드시 먼저 로드한다.
스킬 파일을 읽지 않고 작업에 착수하는 것을 금지한다.

| 상황 | 사용할 스킬 |
|------|------------|
| AGENTS.md만 존재하는 빈 프로젝트를 **Express.js + DDD + API** 구조로 초기화할 때 | `express-api-starter` 스킬 |
| DDD 아키텍처 기반 패키지의 AGENTS.md를 생성하거나 수정할 때 | `ddd-architecture` 스킬 |
| 구현 완료 후 테스트 코드 작성 방식이 **Gemini CLI 위임**으로 설정된 경우 | `gemini-test-delegate` 스킬 |

### express-api-starter 스킬 트리거 조건
- `AGENTS.md`(또는 `RULES.md`)만 존재하고 `src/` 또는 `apps/` 디렉터리가 없는 상태에서
- 사용자가 "Express", "express-api", "API 서버 초기화", "프로젝트 세팅" 등을 언급할 때

### ddd-architecture 스킬 트리거 조건
- API 패키지 AGENTS.md 생성·수정 요청이 있을 때
- DDD, Domain-Driven Design, 레이어드 아키텍처, 헥사고날 아키텍처 등의 키워드가 등장할 때
- `modules/{module}/domain|application|infrastructure` 구조를 다루는 모든 작업
```

---

## [A:PKG] OVERVIEW — 설명이 있을 때

```markdown
## OVERVIEW

{패키지 역할 2~3문장}
```

---

## [A:PKG] STRUCTURE: API 앱

```markdown
## STRUCTURE

```
src/
├── modules/{module}/
│   ├── domain/
│   │   ├── {module}.repository.interface.ts
│   │   └── {module}.validation.ts
│   ├── application/
│   │   └── {module}.service.ts
│   └── infrastructure/
│       ├── http/
│       │   ├── {module}.controller.ts
│       │   └── {module}.route.ts
│       └── persistence/
│           └── {module}.repository.ts
├── shared/
├── config.ts
└── index.ts
```

> DDD 레이어 책임·의존성 방향·핵심 규칙: `RULES.md#api`
```

---

## [A:PKG] STRUCTURE: Frontend 앱

```markdown
## STRUCTURE

```
src/
├── components/{component}/
│   ├── {component}.tsx
│   └── index.ts
├── pages/ (또는 app/)
├── hooks/
├── stores/
├── lib/
└── types/
```

> 컴포넌트 규칙·hooks 패턴: `RULES.md#frontend`
```

---

## [A:PKG] STRUCTURE: CLI 앱

```markdown
## STRUCTURE

```
src/
├── commands/
├── prompts/
├── lib/
└── index.ts
```

> CLI 규칙·exit code: `RULES.md#cli`
```

---

## [A:PKG] STRUCTURE: 공유 라이브러리

```markdown
## STRUCTURE

```
src/
├── {domain}/
└── index.ts   # 유일한 공개 진입점
```

> export 규칙·의존성 방향: `RULES.md#shared`
```
