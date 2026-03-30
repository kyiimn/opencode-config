# AGENTS.md 템플릿 레퍼런스

이 파일은 AGENTS.md 생성 시 채워 넣을 섹션 구조와 예시를 담고 있습니다.
인터뷰 답변을 바탕으로 해당 섹션을 선택·조합하여 최종 파일을 완성합니다.

섹션 태그:
- `[ROOT]` — 루트 AGENTS.md 전용
- `[PACKAGE]` — 패키지/앱별 AGENTS.md 전용
- `[COMMON]` — 양쪽 모두 사용 가능

---

## [ROOT][COMMON] HEADER — 항상 포함

```markdown
# {PROJECT_NAME} — AGENTS.md

**Stack:** TypeScript (strict) · {STACK_SUMMARY}
**Package Manager:** {npm | pnpm}
```

---

## [PACKAGE] HEADER — 패키지 AGENTS.md 항상 포함

```markdown
# {PACKAGE_PATH} — AGENTS.md

> 루트 [`/AGENTS.md`](/AGENTS.md)의 전역 규칙을 상속합니다.
> 이 파일은 `{PACKAGE_PATH}` 패키지에 특화된 규칙만 기술합니다.

**패키지 역할:** {패키지가 담당하는 책임 한 줄 요약}
**패키지 타입:** {API 서버 앱 | Frontend 앱 | CLI 앱 | 공유 라이브러리}
```

---

## [ROOT] OVERVIEW — 프로젝트 설명이 있을 때

```markdown
## OVERVIEW

{프로젝트 설명을 2~4문장으로 요약}
```

---

## [ROOT] STRUCTURE — 싱글 프로젝트

```markdown
## PROJECT STRUCTURE

```
{project-name}/
├── src/
│   ├── index.ts          # 진입점
│   ├── {domain}/         # 도메인별 디렉터리
│   └── types/            # 공유 타입
├── tests/
├── package.json
└── tsconfig.json
```
```

---

## [ROOT] STRUCTURE — 모노레포 (pnpm)

```markdown
## PROJECT STRUCTURE

pnpm workspaces 기반 모노레포입니다.

```
{project-name}/
├── apps/
│   ├── {cli}/            # CLI 애플리케이션 (해당 시)
│   ├── {web}/            # Frontend 애플리케이션 (해당 시)
│   └── {api}/            # API 서버 애플리케이션 (해당 시)
├── packages/
│   └── core/             # 공유 비즈니스 로직 및 타입
├── pnpm-workspace.yaml
├── package.json          # 루트 (devDependencies, scripts)
└── tsconfig.base.json    # 공통 TS 설정
```

### 디렉터리 역할 구분
- `apps/` — 실행 가능한 애플리케이션 (배포 단위)
- `packages/` — 앱 간 공유되는 라이브러리 (core, ui, config 등)

### 패키지 간 임포트 규칙
- 워크스페이스 패키지는 항상 패키지 이름으로 임포트: `import { X } from '@{scope}/core'`
- 상대 경로로 패키지/앱 경계를 넘지 않는다
```

---

## [ROOT] CODE STANDARDS — 항상 포함

```markdown
## CODE STANDARDS

### TypeScript
- **strict mode 필수** (`"strict": true` in tsconfig)
- `any` 사용 금지 — `unknown` 후 타입 가드 사용
- 명시적 반환 타입 선언 권장 (`noImplicitReturns: true`)
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
- **번들링이 필요한 경우 반드시 Vite 사용** (`vite build`)
- **TypeScript 파일 직접 실행 시 vite-node 사용** (`vite-node src/index.ts`)
- tsc는 타입 검사 전용 (`tsc --noEmit`) — 빌드 도구로 사용 금지
- esbuild, webpack, ts-node, tsx 등 다른 번들러/실행기 도입 금지

### Do
- 함수형으로 작성, 단일 책임 원칙 준수
- `async/await` 사용 (콜백 패턴 금지)
- 에러는 반드시 상위 레이어로 throw (각 레이어에서 삼키지 않음)
- 새 파일 작성 전 `ARCHITECTURE` 섹션의 기존 파일을 먼저 확인
- 타입은 최대한 좁게 (union, literal type 적극 활용)

### Don't
- `console.log` 프로덕션 코드에 사용 금지 (logger 모듈 사용)
- Magic number/string 금지 — 상수로 추출
- `// TODO` 없이 커밋 금지
- 새 외부 패키지 추가 시 사용자 승인 없이 설치 금지
- `any` 타입 사용 금지 — `unknown` 후 타입 가드 사용
```

---

## [ROOT] ARCHITECTURE — 핵심 파일 경로가 있을 때

에이전트가 리포지토리 탐색 단계를 건너뛸 수 있도록 핵심 파일 위치를 명시한다.
사용자가 제공한 경로를 실제 값으로 채워 넣는다.

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

> 위 파일들을 직접 수정하기 전 반드시 전체 내용을 읽는다.
```

---

## [ROOT] COMMANDS — 항상 포함

```markdown
## COMMANDS

| 목적 | 명령어 | 실행 위치 |
|------|--------|----------|
| 타입 검사 | `pnpm tsc --noEmit` | 패키지 디렉터리 |
| 테스트 (단일) | `pnpm vitest run {파일경로}` | 패키지 디렉터리 |
| 테스트 (전체) | `pnpm vitest run` | 루트 |
| 린트 | `pnpm eslint --fix {파일경로}` | 루트 |
| 빌드 | `pnpm build` | **명시적 요청 시에만** |

> 타입 검사와 테스트는 구현 완료 후 자율적으로 실행한다.
> 빌드와 배포는 반드시 사용자 승인 후 실행한다.
```

---

## [ROOT] CODE EXAMPLES — Good/Bad 파일 참조가 있을 때

```markdown
## CODE EXAMPLES

### 참고할 파일 (Good)
- {예: `src/components/UserCard.tsx`} — 컴포넌트 작성 패턴
- {예: `src/hooks/useData.ts`} — 데이터 페칭 훅 패턴
- {예: `src/modules/user/application/user.service.ts`} — Service 레이어 패턴

### 참고 금지 파일 (Legacy / Bad)
- {예: `src/legacy/OldAdmin.tsx`} — 클래스 기반, 현재 패턴과 다름
- {예: `src/utils/deprecated.ts`} — 삭제 예정, 임포트 금지
```

---

## [ROOT] SAFETY & PERMISSIONS — 항상 포함

```markdown
## SAFETY & PERMISSIONS

### 승인 없이 자율 실행 가능
- 파일 읽기, 목록 조회
- 타입 검사 (`tsc --noEmit`)
- 린트, 포맷 (`eslint`, `prettier`)
- 단일 테스트 파일 실행

### 반드시 사용자 승인 후 실행
- 패키지 설치 / 삭제 (`pnpm add`, `pnpm remove`)
- `git push`, 브랜치 생성, PR 생성
- 파일 또는 디렉터리 삭제
- 전체 빌드 / 배포
- 환경변수 파일(`.env`) 수정
```

---

## [ROOT] CLI 툴 — CLI 포함 시

```markdown
## CLI

- 진입점: `apps/cli/src/index.ts` (모노레포) 또는 `src/cli/index.ts`
- 파서: Commander.js 또는 yargs 사용
- 커맨드 파일 위치: `src/commands/{command-name}.ts`
- 각 커맨드는 단일 책임 원칙 준수 (파싱 → 비즈니스 로직 위임 → 출력)
- exit code: 성공 `0`, 사용자 오류 `1`, 시스템 오류 `2`
- stderr로 오류 출력, stdout으로 정상 결과 출력
```

---

## [ROOT] FRONTEND (React 19) — Frontend 포함 시

```markdown
## FRONTEND (React 19)

- 위치: `apps/web/src` (모노레포) 또는 `src/`
- 컴포넌트 파일명: **{FILE_NAMING_CONVENTION}** (예: `{COMPONENT_FILE_EXAMPLE}`)
- 컴포넌트 함수명: PascalCase (`UserCard`, `DashboardLayout`)
- hooks: `use` prefix camelCase (`useUserData`, `useAuthState`)
- 상태 관리: [Zustand | Jotai | Context API] (프로젝트 설정에 따라)
- 스타일: [Tailwind CSS | CSS Modules | styled-components]

### 컴포넌트 구조 원칙
- Server Component vs Client Component 구분 명시 (`"use client"` 위치)
- 단일 파일 내 export는 하나의 주 컴포넌트만 (index.ts 배럴 파일 허용)
- props 타입은 인라인 또는 동일 파일 상단에 정의: `type Props = { ... }`
- React 19 Actions 사용 시 `useActionState`, `useFormStatus` 활용

### 금지 패턴
- `useEffect`로 데이터 페칭 금지 — React Query / SWR / Server Action 사용
- `any` 타입의 props 금지
```

---

## [ROOT] API 서버 — API 포함 시

```markdown
## API SERVER

- 위치: `apps/api/src` (모노레포) 또는 `src/`
- 프레임워크: {Hono | Express | Fastify | NestJS}
- 포트 설정: 환경변수 `PORT` (기본값 `3000`)
- ORM: Prisma
- 유효성 검증: Zod

### DDD Layer Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP Layer (infrastructure/http)          │
│  Routes → Controllers (arrow functions for `this` binding)  │
├─────────────────────────────────────────────────────────────┤
│                  Application Layer                           │
│  Services - Business logic, orchestration                    │
├─────────────────────────────────────────────────────────────┤
│                       Domain Layer                           │
│  Interfaces (Repository), Validation Schemas (Zod)           │
│  DTOs, Entities                                              │
├─────────────────────────────────────────────────────────────┤
│              Persistence Layer (infrastructure/persistence)  │
│  Repository implementations (Prisma)                         │
└─────────────────────────────────────────────────────────────┘
```

#### 레이어별 책임

| Layer | Directory | Responsibility |
|-------|-----------|----------------|
| HTTP | `infrastructure/http/` | Routes, Controllers, Middleware |
| Application | `application/` | Business logic, use case orchestration |
| Domain | `domain/` | Repository interfaces, DTOs, Entities, Validation schemas |
| Persistence | `infrastructure/persistence/` | Repository implementations (Prisma) |

#### 의존성 방향

```
HTTP Layer → Application Layer → Domain Layer ← Persistence Layer
```

- **Domain Layer**: 다른 레이어에 의존하지 않음 — 인터페이스만 정의
- **Persistence Layer**: Domain 인터페이스를 구현 (Prisma 사용)
- **Application Layer**: Domain 인터페이스에 의존 (구현체에 직접 의존 금지)
- **HTTP Layer**: Application Services에만 의존

### 모듈 구조 패턴

각 도메인 모듈은 아래 구조를 일관되게 따른다:

```
src/modules/{module}/
├── domain/
│   ├── {module}.repository.interface.ts  # Repository 인터페이스 + DTOs + Entities
│   └── {module}.validation.ts            # Zod validation schemas
├── application/
│   └── {module}.service.ts               # Business logic
└── infrastructure/
    ├── http/
    │   ├── {module}.controller.ts        # Request handlers (arrow function 메서드)
    │   └── {module}.route.ts             # Router 정의
    └── persistence/
        └── {module}.repository.ts        # Prisma 구현체
```

### 규칙
- **Controller 메서드는 반드시 arrow function** — `this` 바인딩 문제 방지
  ```typescript
  // ✅ 올바른 방식
  getUser = async (req: Request, res: Response) => { ... }
  // ❌ 금지
  async getUser(req: Request, res: Response) { ... }
  ```
- **Application Layer는 Repository 인터페이스만 주입받는다** (구현체 직접 참조 금지)
- **Zod schema는 Domain Layer에 위치** — Controller에서 직접 정의 금지
- **Entity/DTO 분리**: DB 모델(Entity)과 외부 노출 타입(DTO)을 구분
- HTTP 상태 코드 일관성: 200(OK), 201(Created), 400(Bad Request), 401, 403, 404, 422(Validation), 500
- 에러 응답 형식 통일: `{ error: string, code?: string, details?: unknown }`
- 환경변수는 `src/config.ts`에서 중앙 관리 (Zod로 검증)
- 데이터베이스 트랜잭션은 Application(Service) 레이어에서 관리
```

---

## [COMMON] TESTING — 항상 포함

```markdown
## TESTING

- **테스트 러너: Vitest 필수** — Jest 등 다른 테스트 프레임워크 사용 금지
- 테스트 파일 위치: 소스 파일과 같은 디렉터리 (`{name}.test.ts`) 또는 `tests/`
- 모킹: `vi.mock()`, `vi.fn()`, `vi.spyOn()` 사용
- 커버리지: `@vitest/coverage-v8` (`vitest run --coverage`)
- 커버리지 목표: 핵심 비즈니스 로직 80%+
- 테스트 구조: `describe > it` (given/when/then 주석 권장)
- 설정 파일: `vitest.config.ts` (Vite 설정과 공유 가능 — `defineConfig` 재사용)
```

---

## [ROOT] FILE REFERENCES — 항상 포함

just-in-time 로딩 원칙: 모든 규칙을 AGENTS.md에 나열하는 대신,
상세 문서는 이 섹션에 경로만 적어 두고 에이전트가 필요할 때 읽도록 유도한다.

```markdown
## FILE REFERENCES

아래 파일 경로(`@` 접두사)를 만나면 해당 파일을 Read 툴로 로드한다.
현재 태스크와 관련된 경우에만 읽는다.

| 문서 | 경로 |
|------|------|
| API 설계 가이드 | `@docs/api-guide.md` |
| DB 스키마 | `@prisma/schema.prisma` |
| 환경변수 목록 | `@.env.example` |
| 테스트 작성 가이드 | `@docs/testing-guide.md` |

> 이 목록은 프로젝트에 맞게 수정한다. 없는 파일은 삭제한다.
```

**생성 시 주의**: 사용자가 제공한 파일 경로가 있으면 실제 경로로 교체하고,
없으면 위 기본 목록에서 프로젝트에 해당하는 항목만 남긴다.

---

## [ROOT] GIT — 항상 포함

```markdown
## GIT CONVENTIONS

- 브랜치: `feature/{task}`, `fix/{issue}`, `chore/{task}`
- 커밋 메시지: Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`)
- PR 전 `tsc --noEmit` 및 lint 통과 필수
```

---

## [COMMON] AGENT WORKFLOW — 항상 포함

```markdown
## AGENT WORKFLOW

모든 기능 구현은 **계획 → 구현 → 검증** 루프를 검증이 통과할 때까지 반복한다.

```
┌─────────────────────────────────────────────┐
│  1. 계획: 역할·입력·출력 명세 작성 + 승인   │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  2. 구현: 승인된 명세 범위 내에서만 작성    │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  3. 검증: 테스트 작성 및 실행               │
└──────────┬──────────────────┬───────────────┘
           │ PASS             │ FAIL
           ▼                  ▼
        완료            부족한 부분 파악
                        → 1. 계획으로 돌아감
```

### 1단계: 계획 (구현 전 필수)

코드를 작성하기 전에 추가하거나 수정할 코드의 명세를 아래 형식으로 먼저 정의한다.
계획 없이 바로 구현에 착수하는 것을 금지한다.

```
## 구현 계획

### [추가 / 수정] {파일 경로}
- **역할**: 이 코드가 담당하는 책임 (한 문장)
- **입력**: 함수/메서드가 받는 파라미터와 타입
- **출력**: 반환값 또는 사이드이펙트 (반환 타입 명시)
- **변경 이유**: 기존 코드 수정 시, 변경이 필요한 이유

(수정 대상 파일마다 반복)
```

계획을 사용자에게 제시하고 승인을 받은 후 구현을 시작한다.

### 2단계: 구현

승인된 계획의 명세(역할·입력·출력)를 벗어나는 구현을 금지한다.
명세 변경이 필요하면 구현을 멈추고 1단계로 돌아가 계획을 재수립한다.

### 3단계: 검증 (구현 후 필수)

구현이 완료되면 반드시 아래 순서로 검증 파이프라인을 실행한다.
테스트 없이 작업을 완료로 보고하는 것을 금지한다.

**검증 파이프라인 (순서 엄수)**
```bash
# Step 1: 타입 검사 — 컴파일 에러를 먼저 제거
pnpm tsc --noEmit

# Step 2: 테스트 실행
pnpm vitest run {파일}   # 구현한 파일 대상 먼저
pnpm vitest run          # 전체 회귀 확인
```

**자동화 피드백 루프 (인간 개입 없이 자가 처리)**

에러가 발생하면 사람이 터미널 로그를 복사해줄 때까지 기다리지 않는다.
에이전트 스스로 아래 파이프라인을 실행한다:

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

**테스트 작성 기준**
- 구현한 함수/메서드마다 정상 케이스(happy path) 최소 1개 작성
- 경계값·예외 케이스(edge case, error path)를 식별하여 함께 작성
- 테스트 케이스 이름은 `{대상}_{조건}_{기대결과}` 형식으로 작성
  - 예: `getUser_존재하지_않는_id_NotFoundError반환`

**검증 실패 시 처리 (루프 재진입)**
단순히 코드를 수정하고 재실행하는 것을 금지한다. 반드시 아래 순서를 따른다:

1. **실패 원인 분석**: 어떤 가정이 틀렸는지, 어느 레이어의 명세가 잘못됐는지 파악
2. **1단계(계획)로 복귀**: 잘못된 명세를 수정한 새 계획을 작성하고 승인을 받는다
3. **2단계(구현) 재실행**: 수정된 계획에 따라 다시 구현한다
4. **3단계(검증) 재실행**: 파이프라인 전체를 처음부터 다시 실행한다

이 루프는 모든 테스트가 통과할 때까지 반복한다.
```

---

## [COMMON] OBSERVABILITY — 항상 포함

```markdown
## OBSERVABILITY

에이전트는 모든 작업 세션에서 아래 추적 정보를 구조화된 형식으로 기록한다.
무한 루프·프롬프트 오해·툴 에러를 사후에 추적할 수 있어야 한다.

### 세션 트레이스 형식

작업을 시작할 때 다음 헤더를 출력한다:

```
## TRACE [{ISO 타임스탬프}] {작업 제목}
- Loop: 0 / 계획 수립 중
```

루프를 반복할 때마다 카운터를 증가시켜 기록한다:

```
## TRACE [{타임스탬프}] Loop {N}
- 단계: {계획|구현|검증}
- 상태: {진행중|PASS|FAIL}
- 실패 원인 (FAIL 시):
    - 에러 유형: {TypeScript타입에러|Vitest실패|런타임에러|툴에러}
    - 에러 위치: {파일명:라인} 또는 {describe > it 경로}
    - 핵심 메시지: {에러 메시지 한 줄 요약}
    - 근본 원인 판단: {프롬프트오해|명세오류|구현버그|툴장애} 중 하나
```

### 루프 상한 및 탈출 조건

- **루프 상한**: 동일 작업에서 계획→구현→검증 루프가 **3회** 를 초과하면 자동 진행을 중단한다.
- **탈출 시 보고 형식**:

```
## TRACE [중단] Loop {N} — 루프 상한 초과

반복 실패 요약:
- Loop 1 실패: {근본 원인}
- Loop 2 실패: {근본 원인}
- Loop 3 실패: {근본 원인}

패턴 분석: {동일 원인 반복인지 / 원인이 매번 다른지}
추정 원인: {프롬프트 오해 | 명세 자체의 모순 | 환경/툴 문제}
권장 조치: {사용자에게 제시할 다음 행동}
```

중단 후 사용자의 판단을 기다린다. 임의로 계속 진행하지 않는다.

### 툴 에러 처리

명령어 실행 실패(exit code ≠ 0)와 에이전트 로직 실패를 구분한다:

| 에러 유형 | 판단 기준 | 처리 방법 |
|-----------|-----------|-----------|
| TypeScript 타입 에러 | `tsc` exit code 1 + 타입 에러 메시지 | 자동화 피드백 루프로 재진입 |
| Vitest 실패 | `vitest` exit code 1 + 테스트 실패 | 자동화 피드백 루프로 재진입 |
| 툴 장애 | 명령어 자체가 실행 불가 / 타임아웃 | 즉시 중단 후 사용자에게 보고 |
| 환경 문제 | 동일 에러가 수정 없이 반복 | 즉시 중단 후 사용자에게 보고 |

툴 장애와 환경 문제는 코드 수정으로 해결할 수 없으므로 루프를 돌리지 않는다.
```

---

## [COMMON] LIVING DOCUMENT (Mitchell Hashimoto 원칙) — 항상 포함

```markdown
## LIVING DOCUMENT

> "에이전트가 실수할 때마다 AGENTS.md에 규칙을 추가한다." — Mitchell Hashimoto

AGENTS.md는 한 번 작성하고 끝나는 문서가 아니라, 에이전트의 실수로부터 학습하며 지속적으로 진화하는 살아있는 문서다.

### 규칙 추가 트리거

아래 상황이 발생하면 즉시 이 파일에 규칙을 추가한다:

- 에이전트가 같은 실수를 두 번 이상 반복했을 때
- 검증 루프가 2회 이상 실패했을 때 (반복 실패의 패턴이 보일 때)
- 에이전트가 명시되지 않은 판단을 내려 예상과 다른 결과를 냈을 때
- 사용자가 구현 결과를 보고 방향을 수정해야 했을 때

### 규칙 추가 방법

```
## [날짜] 추가된 규칙: {한 줄 요약}
- **배경**: 어떤 실수 또는 상황에서 이 규칙이 생겼는지
- **규칙**: 앞으로 에이전트가 반드시 따라야 할 행동
```

규칙은 해당하는 섹션(AGENT WORKFLOW, CODE STANDARDS, API SERVER 등)에 직접 추가하거나,
별도의 `## LEARNED RULES` 섹션을 만들어 누적한다.

### 원칙

- 규칙은 구체적으로: "주의한다" (X) → "~할 때는 반드시 ~한다" (O)
- 규칙은 에이전트가 읽는다는 것을 염두에 두고 명령형으로 작성
- 낡은 규칙은 정기적으로 검토하여 불필요한 것은 제거한다
```

---

## [ROOT] OH-MY-OPENCODE ORCHESTRATION — 항상 포함

```markdown
## AGENT ORCHESTRATION (oh-my-opencode / Sisyphus)

이 프로젝트는 oh-my-opencode의 Sisyphus 오케스트레이션을 사용합니다.

### 에이전트 위임 원칙
- **Sisyphus(오케스트레이터)**: 태스크 분해 및 서브에이전트 위임 담당. 직접 코드를 작성하지 않음
- **Hephaestus**: 실제 코드 구현 (파일 수정, 생성)
- **Oracle/Prometheus**: 계획 수립, 아키텍처 검토
- **Librarian/Explore**: 코드베이스 탐색, 문서 조회

### Sisyphus → Hephaestus 핸드오프 규격

Sisyphus가 Hephaestus에게 구현을 위임할 때는 반드시 아래 구조를 마크다운 리스트 형태로 전달한다.
자연어 단독 지시는 금지한다. 컨텍스트 누락으로 인한 방황을 방지하기 위함이다.

```
## 위임: {작업 제목}

- **작업 대상 파일**:
  - `{추가할 파일 경로}` — 신규 생성
  - `{수정할 파일 경로}` — 기존 수정

- **구현 명세**:
  - 역할: {이 코드가 담당하는 책임}
  - 입력: {파라미터명: 타입, ...}
  - 출력: {반환 타입 또는 사이드이펙트}
  - 제약: {레이어 규칙, 금지 패턴 등 구현 시 반드시 지켜야 할 사항}

- **참조할 규칙**:
  - {예: AGENTS.md > DDD ARCHITECTURE > 의존성 방향}
  - {예: AGENTS.md > CODE STANDARDS > Do/Don't}
  - {예: apps/api/AGENTS.md > 모듈 구조 패턴}
```

### AGENTS.md 계층 구조 (모노레포)
각 패키지 루트에 패키지별 `AGENTS.md`를 두어 컨텍스트를 최소화합니다:
- `/AGENTS.md` — 프로젝트 전체 규칙 (이 파일)
- `/packages/core/AGENTS.md` — core 패키지 전용 규칙
- `/apps/{web|api|cli}/AGENTS.md` — 각 앱 전용 규칙

### opencode.json으로 규칙 파일 분리 (선택)
단일 AGENTS.md 대신 여러 파일로 분리하려면 `.opencode/opencode.json`을 사용합니다:
```json
{
  "instructions": [
    "docs/development-standards.md",
    "docs/testing-guidelines.md",
    "apps/*/AGENTS.md",
    "packages/*/AGENTS.md"
  ]
}
```

### 에이전트에게 작업 지시 시 권장 패턴
```
@Sisyphus {기능 설명}: {구체적 요구사항}
```
복잡한 작업 시작 전 `/start-work`로 Prometheus 인터뷰를 먼저 실행하세요.
```

---

## 조합 가이드

### 루트 AGENTS.md

| 조건 | 포함 섹션 |
|------|----------|
| 항상 | HEADER[ROOT], COMMANDS, CODE STANDARDS, SAFETY & PERMISSIONS, AGENT WORKFLOW, OBSERVABILITY, LIVING DOCUMENT, TESTING, FILE REFERENCES, GIT, ORCHESTRATION |
| 프로젝트 설명 있음 | OVERVIEW |
| 싱글 프로젝트 | STRUCTURE (싱글) |
| 모노레포 | STRUCTURE (모노레포) |
| 핵심 파일 경로 제공 | ARCHITECTURE |
| Good/Bad 파일 제공 | CODE EXAMPLES |
| CLI 포함 | CLI 툴[ROOT] |
| Frontend 포함 | FRONTEND[ROOT] |
| API 포함 | API 서버[ROOT] |

### 패키지 AGENTS.md

| 조건 | 포함 섹션 |
|------|----------|
| 항상 | HEADER[PACKAGE], AGENT WORKFLOW, OBSERVABILITY, LIVING DOCUMENT, TESTING |
| 패키지 설명 있음 | OVERVIEW[PACKAGE] |
| API 앱 | STRUCTURE[PKG-API], API 규칙[PACKAGE] |
| Frontend 앱 | STRUCTURE[PKG-WEB], Frontend 규칙[PACKAGE] |
| CLI 앱 | STRUCTURE[PKG-CLI], CLI 규칙[PACKAGE] |
| 공유 라이브러리 | STRUCTURE[PKG-CORE], 공유 라이브러리 규칙[PACKAGE] |

> 패키지 AGENTS.md에는 루트의 전역 규칙(TypeScript strict, 파일명, Git, Orchestration 등)을
> **절대 반복하지 않는다.** 패키지 고유의 디렉터리 구조와 규칙만 기술한다.

---

# ══════════════════════════════════════
# [PACKAGE] 섹션 모음
# ══════════════════════════════════════

---

## [PACKAGE] OVERVIEW — 패키지 설명이 있을 때

```markdown
## OVERVIEW

{패키지 역할을 2~3문장으로 요약}
```

---

## [PACKAGE] STRUCTURE: API 서버 앱 (apps/api)

```markdown
## STRUCTURE

```
src/
├── modules/          # 도메인 모듈 (아래 모듈 구조 패턴 참고)
│   └── {module}/
│       ├── domain/
│       │   ├── {module}.repository.interface.ts
│       │   └── {module}.validation.ts
│       ├── application/
│       │   └── {module}.service.ts
│       └── infrastructure/
│           ├── http/
│           │   ├── {module}.controller.ts
│           │   └── {module}.route.ts
│           └── persistence/
│               └── {module}.repository.ts
├── shared/           # 공통 미들웨어, 에러 핸들러, 유틸
├── config.ts         # 환경변수 중앙 관리 (Zod 검증)
└── index.ts          # 서버 진입점
```
```

---

## [PACKAGE] API 규칙 — API 앱 전용

```markdown
## DDD ARCHITECTURE

### 레이어 의존성 방향
```
HTTP Layer → Application Layer → Domain Layer ← Persistence Layer
```

### 레이어별 책임
| Layer | Directory | Responsibility |
|-------|-----------|----------------|
| HTTP | `infrastructure/http/` | Routes, Controllers, Middleware |
| Application | `application/` | Business logic, use case orchestration |
| Domain | `domain/` | Repository interfaces, DTOs, Entities, Zod schemas |
| Persistence | `infrastructure/persistence/` | Prisma 구현체 |

### 핵심 규칙
- **Controller 메서드는 반드시 arrow function** (`this` 바인딩 문제 방지)
  ```typescript
  // ✅ 올바른 방식
  getUser = async (req: Request, res: Response) => { ... }
  // ❌ 금지
  async getUser(req: Request, res: Response) { ... }
  ```
- **Application Layer는 Repository 인터페이스만 주입** (구현체 직접 참조 금지)
- **Zod schema는 Domain Layer에 위치** (Controller 내 직접 정의 금지)
- **Entity/DTO 분리**: DB 모델(Entity)과 외부 노출 타입(DTO) 구분
- HTTP 상태 코드: 200, 201, 400, 401, 403, 404, 422, 500
- 에러 응답 형식: `{ error: string, code?: string, details?: unknown }`
- DB 트랜잭션은 Application(Service) 레이어에서 관리
```

---

## [PACKAGE] STRUCTURE: Frontend 앱 (apps/web)

```markdown
## STRUCTURE

```
src/
├── components/       # 재사용 UI 컴포넌트
│   └── {component}/
│       ├── {component}.tsx
│       └── index.ts
├── pages/ (또는 app/)  # 라우트/페이지 단위 컴포넌트
├── hooks/            # 커스텀 훅 (use*.ts)
├── stores/           # 전역 상태 (Zustand / Jotai)
├── lib/              # 유틸, API 클라이언트
└── types/            # 프론트엔드 전용 타입
```
```

---

## [PACKAGE] Frontend 규칙 — Frontend 앱 전용

```markdown
## FRONTEND RULES

- 컴포넌트 함수명: PascalCase (`UserCard`, `DashboardLayout`)
- hooks: `use` prefix camelCase (`useUserData`, `useAuthState`)
- props 타입: 동일 파일 상단에 `type Props = { ... }` 정의
- Server Component / Client Component 경계 명시 (`"use client"` 최소화)
- 단일 파일 내 주 컴포넌트 export 하나만 (배럴 index.ts 허용)
- React 19 Actions 사용 시 `useActionState`, `useFormStatus` 활용

### 금지 패턴
- `useEffect`로 데이터 페칭 금지 — React Query / SWR / Server Action 사용
- `any` 타입 props 금지
```

---

## [PACKAGE] STRUCTURE: CLI 앱 (apps/cli)

```markdown
## STRUCTURE

```
src/
├── commands/         # 커맨드 파일 ({command-name}.command.ts)
├── prompts/          # 인터랙티브 입력 처리
├── lib/              # 공통 유틸
└── index.ts          # CLI 진입점 (Commander.js / yargs)
```
```

---

## [PACKAGE] CLI 규칙 — CLI 앱 전용

```markdown
## CLI RULES

- 커맨드마다 단일 책임 (파싱 → 비즈니스 로직 위임 → 출력)
- exit code: 성공 `0`, 사용자 오류 `1`, 시스템 오류 `2`
- 오류 메시지: stderr 출력 / 정상 결과: stdout 출력
- 긴 작업에는 반드시 진행 표시 (ora 등) 사용
```

---

## [PACKAGE] STRUCTURE: 공유 라이브러리 (packages/*)

```markdown
## STRUCTURE

```
src/
├── {domain}/         # 기능 도메인별 디렉터리
└── index.ts          # 공개 API 진입점 (명시적 re-export만)
```
```

---

## [PACKAGE] 공유 라이브러리 규칙 — packages/* 전용

```markdown
## SHARED LIBRARY RULES

- **`src/index.ts`가 유일한 공개 진입점** — 내부 경로 직접 임포트 금지
  ```typescript
  // ✅ 올바른 방식
  import { UserSchema } from '@{scope}/core'
  // ❌ 금지
  import { UserSchema } from '@{scope}/core/src/user/schema'
  ```
- **이 패키지에서 앱 패키지(`apps/*`)를 절대 임포트하지 않는다**
- 외부로 노출하지 않을 내부 구현은 export하지 않는다
- Breaking change 발생 시 반드시 semver major 버전 업
```

