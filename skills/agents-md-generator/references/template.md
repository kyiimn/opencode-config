# 템플릿 레퍼런스

이 파일은 `AGENTS.md`와 `RULES.md` 두 파일을 생성할 때 사용하는 섹션 모음입니다.

## 파일 역할 구분

| 파일 | 역할 | 읽는 시점 |
|------|------|-----------|
| `AGENTS.md` | 행동 지침 — 무엇을 언제 어떻게 할지 | **매 세션** 자동 주입 |
| `RULES.md` | 상세 레퍼런스 — 규칙의 구체적 정의·예시 | **필요한 순간** 에이전트가 직접 로드 |

**핵심 원칙**: AGENTS.md는 포인터만 갖는다. 상세 규칙은 모두 RULES.md에 있다.

## 섹션 태그

- `[A]` — AGENTS.md에 포함 (루트·패키지 공통)
- `[A:ROOT]` — AGENTS.md 루트 전용
- `[A:PKG]` — AGENTS.md 패키지 전용
- `[R]` — RULES.md에만 포함

---

# ════════════════════════════════════════
# AGENTS.md 섹션
# ════════════════════════════════════════

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
├── apps/
│   ├── {cli}/
│   ├── {web}/
│   └── {api}/
├── packages/
│   └── core/
├── AGENTS.md
├── RULES.md
├── pnpm-workspace.yaml
├── package.json
└── tsconfig.base.json
```

- `apps/` — 배포 단위 애플리케이션
- `packages/` — 앱 간 공유 라이브러리

**임포트 규칙**: 패키지 이름으로 임포트. 상대 경로로 경계를 넘지 않는다.
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

모든 구현은 **계획 → 구현 → 검증 → 리팩터링** 루프를 완료해야 끝난다.

```
┌───────────────────────────────────────────┐
│  1. 계획: 역할·입력·출력 명세 + 사용자 승인 │
└──────────────────┬────────────────────────┘
                   ▼
┌───────────────────────────────────────────┐
│  2. 구현: 코드 + JSDoc 동시 작성           │
│     ↳ 상세: RULES.md#jsdoc               │
└──────────────────┬────────────────────────┘
                   ▼
┌───────────────────────────────────────────┐
│  3. 검증: tsc → vitest (자동 피드백 루프)  │
└──────────┬──────────────────┬─────────────┘
           │ PASS             │ FAIL
           ▼                  ▼
┌──────────────────┐    실패 원인 분석
│  4. 리팩터링     │    → 1단계로 복귀
│  ↳ RULES.md#refactoring │
└──────────┬───────┘
           │ 완료
           ▼
        작업 종료
```

### 1단계: 계획

```
### [추가 / 수정] {파일 경로}
- 역할: {한 문장}
- 입력: {파라미터: 타입}
- 출력: {반환 타입 또는 사이드이펙트}
- 변경 이유: {수정 시만 기재}
```
계획을 제시하고 승인 후 구현 시작. 계획 없이 착수 금지.

### 2단계: 구현

구현 코드와 JSDoc을 동시에 작성한다. 어느 한쪽만 작성하는 것을 금지한다.
상세 JSDoc 규칙: `RULES.md#jsdoc`

### 3단계: 검증 (자동 피드백 루프)

**검증 파이프라인 (순서 엄수)**
```bash
pnpm tsc --noEmit          # Step 1: 타입 에러 먼저 제거
pnpm vitest run {파일}     # Step 2: 단일 파일
pnpm vitest run            # Step 3: 전체 회귀
```

에러 발생 시 사람의 개입 없이 자동 처리:

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
    ├─ Vitest 실패           → describe/it 경로 + expect 실패값 파싱
    └─ 런타임/툴 에러        → stderr 전문 + exit code 기록
    │
    ▼
원인 분석 (어느 레이어·파일·가정이 잘못됐는지 특정)
    │
    ▼
검증 실패 처리 → (아래 참고)
```

에러 출력을 파싱할 때는 첫 번째 에러만 보지 않는다.
전체 에러 목록을 수집하여 패턴(동일 파일 반복, 동일 타입 반복)을 파악한 뒤 근본 원인을 하나로 특정한다.

툴장애·환경 문제(명령어 자체 실행 불가, 타임아웃)는 즉시 중단 후 사용자 보고. 루프에 재진입하지 않는다.

**검증 실패 시 처리 (루프 재진입)**

단순 코드 수정 후 재실행 금지. 반드시 아래 순서를 따른다:

1. **실패 원인 분석**: 어떤 가정이 틀렸는지, 어느 레이어의 명세가 잘못됐는지 파악
2. **1단계(계획)로 복귀**: 잘못된 명세를 수정한 새 계획을 작성하고 승인을 받는다
3. **2단계(구현) 재실행**: 수정된 계획에 따라 다시 구현한다
4. **3단계(검증) 재실행**: 파이프라인 전체를 처음부터 다시 실행한다

이 루프는 모든 테스트가 통과할 때까지 반복한다.
루프 3회 초과 시 자동 중단 (OBSERVABILITY 참조).

### 4단계: 리팩터링

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
- 단계: {계획|구현|검증}
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
| 테스트 작성 시 | `RULES.md#testing` |
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

자연어 단독 지시 금지. 반드시 아래 구조로 전달:

```
## 위임: {작업 제목}

- **작업 대상 파일**:
  - `{경로}` — 신규 생성 | 기존 수정

- **구현 명세**:
  - 역할: {책임}
  - 입력: {파라미터명: 타입}
  - 출력: {반환 타입}
  - 제약: {레이어 규칙, 금지 패턴}

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

## [A:PKG] OVERVIEW

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

---

## AGENTS.md 조합 가이드

### 루트

| 조건 | 포함 섹션 |
|------|-----------|
| 항상 | HEADER, COMMANDS, SAFETY & PERMISSIONS, AGENT WORKFLOW, OBSERVABILITY, LIVING DOCUMENT, GIT, FILE REFERENCES, ORCHESTRATION |
| 설명 있음 | OVERVIEW |
| 싱글 프로젝트 | STRUCTURE (싱글) |
| 모노레포 | STRUCTURE (모노레포) |
| 핵심 경로 제공 | ARCHITECTURE |
| Good/Bad 파일 제공 | CODE EXAMPLES |

> 스택별 규칙(CLI/Frontend/API)은 RULES.md에만 기술. AGENTS.md HEADER의 Stack 항목에 한 줄로만 표기.

### 패키지

| 조건 | 포함 섹션 |
|------|-----------|
| 항상 | HEADER[PKG], AGENT WORKFLOW, OBSERVABILITY, LIVING DOCUMENT |
| 설명 있음 | OVERVIEW[PKG] |
| API 앱 | STRUCTURE[PKG-API] |
| Frontend 앱 | STRUCTURE[PKG-WEB] |
| CLI 앱 | STRUCTURE[PKG-CLI] |
| 공유 라이브러리 | STRUCTURE[PKG-CORE] |

---

# ════════════════════════════════════════
# RULES.md 섹션
# ════════════════════════════════════════

---

## [R] HEADER

```markdown
# {PROJECT_NAME} — RULES.md

> 에이전트가 **필요한 순간에만** 로드하는 상세 규칙 레퍼런스.
> 전체를 한 번에 읽지 않는다. 필요한 섹션 앵커를 지정해 해당 부분만 읽는다.

**섹션 목차**
- [`#code-standards`](#code-standards) — TypeScript·네이밍·번들링·Do/Don't
- [`#jsdoc`](#jsdoc) — JSDoc 필수 태그·예시
- [`#api`](#api) — DDD 아키텍처·모듈 구조·API 규칙
- [`#frontend`](#frontend) — React 19 컴포넌트·hooks 규칙
- [`#cli`](#cli) — CLI 커맨드 구조·규칙
- [`#shared`](#shared) — 공유 라이브러리 export·의존성 규칙
- [`#testing`](#testing) — Vitest 설정·커버리지·테스트 작성 기준
- [`#refactoring`](#refactoring) — 코드 스멜 체크리스트·리팩터링 절차
```

---

## [R] CODE STANDARDS {#code-standards}

```markdown
## CODE STANDARDS {#code-standards}

### TypeScript
- **strict mode 필수** (`"strict": true` in tsconfig)
- `any` 사용 금지 — `unknown` 후 타입 가드 사용
- 명시적 반환 타입 선언 (`noImplicitReturns: true`)
- Non-null assertion (`!`) 최소화 — 가드나 옵셔널 체이닝 사용

### 파일명
- **{FILE_NAMING_CONVENTION}** 사용
- 예: {FILE_NAMING_EXAMPLE}

### 네이밍 규칙
| 대상 | 규칙 | 예시 |
|------|------|------|
| 클래스 | PascalCase | `UserService` |
| 인터페이스 | {INTERFACE_PREFIX}PascalCase | `{INTERFACE_EXAMPLE}` |
| Type alias | PascalCase | `UserDto`, `ApiResponse<T>` |
| 함수/메서드 | camelCase | `{FUNCTION_EXAMPLE}` |
| 상수 | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 열거형 | PascalCase (enum) / SCREAMING_SNAKE (값) | `Status.ACTIVE` |

### 번들링 / 실행
- **번들링 필요 시 반드시 Vite** (`vite build`)
- **TS 직접 실행 시 vite-node** (`vite-node src/index.ts`)
- tsc는 타입 검사 전용 (`tsc --noEmit`) — 빌드 도구로 사용 금지
- esbuild, webpack, ts-node, tsx 등 금지

### Do
- 함수형, 단일 책임 원칙
- `async/await` (콜백 금지)
- 에러는 반드시 상위 레이어로 throw
- 타입은 최대한 좁게 (union, literal type 적극 활용)

### Don't
- `console.log` 프로덕션 사용 금지 (logger 사용)
- Magic number/string 금지 — 상수로 추출
- `// TODO` 없이 커밋 금지
- 새 외부 패키지 사용자 승인 없이 설치 금지
- `any` 사용 금지
```

---

## [R] JSDOC {#jsdoc}

```markdown
## JSDOC {#jsdoc}

새로 작성하거나 수정한 **모든 함수·메서드·클래스·인터페이스**에 JSDoc을 반드시 작성한다.
JSDoc 없이 코드를 완료로 보고하는 것을 금지한다.

### 필수 태그

| 대상 | 필수 태그 |
|------|-----------|
| 함수 / 메서드 | 첫 줄 요약, `@param`, `@returns` |
| 비동기 함수 | 위 항목 + `@throws` |
| 클래스 | 첫 줄 요약 |
| 인터페이스 / Type | 첫 줄 요약, 각 프로퍼티 인라인 주석 |

### 작성 예시

```typescript
/**
 * 사용자 ID로 사용자를 조회한다.
 *
 * @param id - 조회할 사용자의 UUID
 * @returns 사용자 엔티티. 존재하지 않으면 null
 * @throws {DatabaseError} DB 연결 실패 시
 */
async getUserById(id: string): Promise<User | null> { ... }

/**
 * 멤버십 등급에 따른 할인율을 계산한다.
 *
 * @param membership - 사용자 멤버십 등급
 * @param total - 원래 주문 금액 (0 이상)
 * @returns 적용 할인율 (0.0 ~ 1.0)
 */
function getDiscountRate(membership: Membership, total: number): number { ... }
```

### 금지 패턴
- `@param value` — 타입·설명 없이 이름만 적는 것 금지
- `// 함수 설명` — 일반 주석으로 JSDoc 대체 금지
- 자명한 getter도 생략 금지 (`getId()`, `getName()` 포함)
```

---

## [R] API {#api}

```markdown
## API {#api}

### DDD Layer Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP Layer (infrastructure/http)          │
│  Routes → Controllers (arrow functions for `this` binding)  │
├─────────────────────────────────────────────────────────────┤
│                  Application Layer                           │
│  Services — Business logic, orchestration                    │
├─────────────────────────────────────────────────────────────┤
│                       Domain Layer                           │
│  Repository interfaces, Zod schemas, DTOs, Entities         │
├─────────────────────────────────────────────────────────────┤
│              Persistence Layer (infrastructure/persistence)  │
│  Repository implementations (Prisma)                         │
└─────────────────────────────────────────────────────────────┘
```

**의존성 방향**: `HTTP → Application → Domain ← Persistence`

| Layer | Directory | Responsibility |
|-------|-----------|----------------|
| HTTP | `infrastructure/http/` | Routes, Controllers, Middleware |
| Application | `application/` | Business logic, use case orchestration |
| Domain | `domain/` | Repository interfaces, DTOs, Entities, Zod schemas |
| Persistence | `infrastructure/persistence/` | Prisma 구현체 |

### 모듈 구조 패턴

```
src/modules/{module}/
├── domain/
│   ├── {module}.repository.interface.ts
│   └── {module}.validation.ts
├── application/
│   └── {module}.service.ts
└── infrastructure/
    ├── http/
    │   ├── {module}.controller.ts
    │   └── {module}.route.ts
    └── persistence/
        └── {module}.repository.ts
```

### 핵심 규칙

- **Controller 메서드는 반드시 arrow function** (`this` 바인딩 문제 방지)
  ```typescript
  // ✅ getUser = async (req, res) => { ... }
  // ❌ async getUser(req, res) { ... }
  ```
- Application Layer는 Repository 인터페이스만 주입 (구현체 직접 참조 금지)
- Zod schema는 Domain Layer에만 위치 (Controller 직접 정의 금지)
- Entity/DTO 분리: DB 모델(Entity) ≠ 외부 노출 타입(DTO)
- HTTP 상태 코드: 200, 201, 400, 401, 403, 404, 422, 500
- 에러 응답: `{ error: string, code?: string, details?: unknown }`
- 환경변수: `src/config.ts` 중앙 관리 (Zod 검증)
- DB 트랜잭션: Application(Service) 레이어에서 관리
```

---

## [R] FRONTEND {#frontend}

```markdown
## FRONTEND {#frontend}

- 컴포넌트 함수명: PascalCase (`UserCard`, `DashboardLayout`)
- hooks: `use` prefix camelCase (`useUserData`, `useAuthState`)
- props 타입: 동일 파일 상단에 `type Props = { ... }`
- Server / Client Component 경계 명시 (`"use client"` 최소화)
- 단일 파일 내 주 컴포넌트 export 하나만 (배럴 index.ts 허용)
- React 19 Actions: `useActionState`, `useFormStatus` 활용

### 금지 패턴
- `useEffect` 데이터 페칭 금지 → React Query / SWR / Server Action 사용
- `any` 타입 props 금지
```

---

## [R] CLI {#cli}

```markdown
## CLI {#cli}

- 커맨드마다 단일 책임 (파싱 → 비즈니스 로직 위임 → 출력)
- exit code: 성공 `0` / 사용자 오류 `1` / 시스템 오류 `2`
- 오류: stderr 출력 / 정상 결과: stdout 출력
- 긴 작업에는 진행 표시 (ora 등) 사용
```

---

## [R] SHARED {#shared}

```markdown
## SHARED LIBRARY {#shared}

- **`src/index.ts`가 유일한 공개 진입점** — 내부 경로 직접 임포트 금지
  ```typescript
  // ✅ import { X } from '@{scope}/core'
  // ❌ import { X } from '@{scope}/core/src/user/schema'
  ```
- `apps/*` 패키지를 절대 임포트하지 않는다
- Breaking change 시 semver major 버전 업
```

---

## [R] TESTING {#testing}

```markdown
## TESTING {#testing}

- **Vitest 필수** — Jest 금지
- 테스트 파일: 소스와 같은 디렉터리 (`{name}.test.ts`) 또는 `tests/`
- 모킹: `vi.mock()`, `vi.fn()`, `vi.spyOn()`
- 커버리지: `@vitest/coverage-v8` (`vitest run --coverage`)
- 커버리지 목표: 핵심 비즈니스 로직 80%+
- 구조: `describe > it` (given/when/then 주석 권장)
- 설정: `vitest.config.ts` (Vite 설정과 공유 가능 — `defineConfig` 재사용)

### 테스트 작성 기준
- 함수·메서드마다 happy path 최소 1개
- edge case·error path 함께 작성
- 테스트명: `{대상}_{조건}_{기대결과}`
  - 예: `getUser_존재하지않는id_NotFoundError반환`
```

---

## [R] REFACTORING {#refactoring}

```markdown
## REFACTORING {#refactoring}

검증(3단계) PASS 후 반드시 실행. PREPARE → IDENTIFY → REFACTOR → VERIFY 순서 준수.

### PREPARE
현재 상태 git commit 후 시작.

### IDENTIFY — 코드 스멜 체크리스트

이번 구현 파일에 한정하여 점검 (해당 없으면 건너뜀):

- [ ] Long Method       — 50줄 초과
- [ ] Duplicated Code   — 동일·유사 로직 반복
- [ ] Large Class       — 단일 책임 초과
- [ ] Long Param List   — 파라미터 3개 초과
- [ ] Magic Number      — 설명 없는 리터럴
- [ ] Nested Conditional— 3단계 이상 중첩 조건문
- [ ] Dead Code         — 미사용 코드·임포트·주석
- [ ] Primitive Obsession — 도메인 개념을 원시 타입으로 표현
- [ ] Feature Envy      — 다른 객체 데이터를 더 많이 사용하는 메서드

### REFACTOR

한 번에 하나의 코드 스멜만 처리. 처리 후 반드시:
```bash
pnpm tsc --noEmit
pnpm vitest run {파일}
```
실패 시 즉시 되돌리고 재시도.

### VERIFY

```bash
pnpm tsc --noEmit && pnpm vitest run
```

### 완료 보고

```
## 리팩터링 완료

처리한 코드 스멜:
- {스멜}: {파일} — {적용 기법}

최종 검증: tsc ✅  vitest {N}개 통과 ✅
```
```
