# 템플릿 — RULES.md 섹션

모든 `[R]` 섹션 모음.
조합 규칙은 `template.md`를 참조한다.

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

### 네이밍 규칙
| 대상 | 규칙 | 예시 |
|------|------|------|
| 파일 | {FILE_NAMING_CONVENTION} | `{FILE_NAMING_EXAMPLE}` |
| 클래스 | {CLASS_NAMING} | `{CLASS_NAMING_EXAMPLE}` |
| 인터페이스 | {INTERFACE_NAMING} | `{INTERFACE_NAMING_EXAMPLE}` |
| Type alias | PascalCase | `UserDto`, `ApiResponse<T>` |
| 함수/메서드 | {METHOD_NAMING} | `{METHOD_NAMING_EXAMPLE}` |
| 상수 | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 열거형 | PascalCase (enum) / SCREAMING_SNAKE (값) | `Status.ACTIVE` |

### 타입 정의 방식
{TYPE_DEFINITION_STYLE}
<!--
  선택지별 실제 출력 예시 (생성 시 해당 항목만 작성):
  - interface 우선:
      객체 형태는 `interface` 사용. 유니온·교차·조건부 타입은 `type` 사용.
      ```typescript
      interface User { id: string; name: string }          // ✅
      type Status = 'active' | 'inactive';                 // ✅
      type alias = interface 를 type으로 대체 금지          // ❌
      ```
  - type 우선:
      모든 타입 정의에 `type alias` 사용. `interface` 사용 금지.
      ```typescript
      type User = { id: string; name: string }             // ✅
      interface User { ... }                               // ❌
      ```
  - 혼용:
      `interface`와 `type`을 상황에 따라 선택. 팀 내 일관성 유지.
-->

### 함수·메서드 선언 방식
{FUNCTION_STYLE}
<!--
  선택지별 실제 출력 예시 (생성 시 해당 항목만 작성):
  - 화살표 함수 우선:
      모든 함수·메서드를 화살표 함수로 선언. `function` 키워드 사용 금지.
      ```typescript
      const getUser = async (id: string): Promise<User> => { ... }   // ✅
      function getUser(id: string) { ... }                            // ❌
      ```
  - function 선언 우선:
      모듈 최상위 함수는 `function` 키워드 사용. 콜백·인라인은 화살표 함수 허용.
      ```typescript
      function getUser(id: string): Promise<User> { ... }            // ✅
      const getUser = async (id: string) => { ... }                  // ❌ (최상위)
      array.map(item => item.id)                                     // ✅ (콜백)
      ```
  - 혼용:
      모듈 최상위·네임드 함수는 `function`. 클래스 메서드·콜백은 화살표 함수.
      ```typescript
      // 최상위
      function bootstrap() { ... }
      // 클래스 메서드 (this 바인딩 보장)
      class Controller { handle = async (req, res) => { ... } }
      ```
-->

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

### 반드시 작성해야 하는 대상

- 함수 / 메서드 (public, protected, private 불문)
- 클래스 / 추상 클래스
- 인터페이스 / 타입 별칭
- 열거형(enum) 및 각 멤버
- 모듈 진입점 (`index.ts` 등)
- 복잡한 로직이 포함된 상수

### 생략 가능한 대상

- 단순 getter / setter (로직 없이 필드만 반환/대입)
- 자명한 생성자 (파라미터를 그대로 필드에 할당만 하는 경우)
- 프레임워크가 자동 생성하는 보일러플레이트 (React 컴포넌트 props 타입 등 이미 타입이 문서화된 경우)

### 필수 태그

```typescript
/**
 * {무엇을 하는지 한 문장 — '~를 반환한다' / '~를 처리한다' 형식}
 *
 * {필요한 경우 상세 설명. 알고리즘, 부수효과, 주의사항 등.}
 *
 * @param {paramName} - {설명}
 * @returns {반환값 설명. void이면 생략 가능}
 * @throws {ErrorType} {발생 조건}
 *
 * @example
 * const result = myFunction('foo');
 * // result === 'bar'
 */
```

- `@param`, `@returns` — 타입은 TypeScript가 보유하므로 타입 표기 생략, 의미 설명만 작성
- `@throws` — 의도적으로 예외를 던지는 경우 반드시 작성
- `@example` — 사용법이 명확하지 않거나 엣지 케이스가 있는 함수에 작성
- `@deprecated` — 더 이상 사용하지 않는 API에 반드시 작성 (대체 API 명시)

### 금지 패턴

```typescript
// ❌ 코드를 그대로 반복하는 주석
/**
 * Gets the user.
 * @param id - The id
 * @returns The user
 */
function getUser(id: string): User { ... }

// ✅ 의미를 설명하는 주석
/**
 * 주어진 ID로 사용자를 조회한다.
 * DB에 존재하지 않으면 null을 반환하며 예외를 던지지 않는다.
 *
 * @param id - 조회할 사용자의 UUID
 * @returns 사용자 객체, 미존재 시 null
 */
function getUser(id: string): User | null { ... }
```

### 생략 가능 예시

```typescript
class Counter {
  private _count = 0;

  // ✅ 단순 getter — JSDoc 생략 가능
  get count(): number { return this._count; }

  // ✅ 단순 setter — JSDoc 생략 가능
  set count(value: number) { this._count = value; }

  /**
   * 카운터를 1 증가시키고 현재 값을 반환한다.
   * count가 MAX_VALUE에 도달하면 0으로 초기화한다.
   *
   * @returns 증가 후의 카운터 값
   */
  increment(): number { ... }
}
```
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

모든 기능 구현은 `/start-work` 명령으로 시작한다.
구현 전 테스트 파일을 먼저 생성하고(Red), 구현 후 통과시키는(Green) TDD 사이클을 따른다.
상세 워크플로우: `.opencode/commands/start-work.md`

### 테스트 파일 규칙

- 위치: 소스 파일과 같은 디렉터리 (`{name}.test.ts`)
- 시나리오 1건 = 테스트 케이스 1건 (1:1 추적)
- 프레임워크: **Vitest 권장** (기존 프로젝트 설정 따름)
- 모킹: `vi.mock()` / `vi.fn()` / `vi.spyOn()`
- 커버리지: `@vitest/coverage-v8` — 핵심 비즈니스 로직 80%+
- 구조: `describe > it` (given/when/then 주석 권장)
- 테스트명: `{대상}_{조건}_{기대결과}`

### 시나리오 파일

위치: `.opencode/test-scenarios/{task-kebab-case}.md`

### 실행 순서

```bash
npm run lint        # 정적 검사
npx tsc --noEmit    # 타입 검사
npm test            # 전체 테스트
```

### 완료 기준

- [ ] 모든 시나리오 항목 `[x PASS]`
- [ ] Lint 경고 0개
- [ ] 타입 에러 0개
- [ ] 기존 테스트 회귀 없음
- [ ] 신규/수정된 함수·클래스·인터페이스에 JSDoc 작성 완료 (`RULES.md#jsdoc` 참조)
```

---

## [R] REFACTORING {#refactoring}

```markdown
## REFACTORING {#refactoring}

검증(4단계) PASS 후 반드시 실행. PREPARE → IDENTIFY → REFACTOR → VERIFY 순서 준수.

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
pnpm eslint --fix {파일경로}
pnpm vitest run {파일}
```
실패 시 즉시 되돌리고 재시도.

### VERIFY

```bash
pnpm tsc --noEmit && pnpm eslint --fix {파일경로} && pnpm vitest run
```

### 완료 보고

```
## 리팩터링 완료

처리한 코드 스멜:
- {스멜}: {파일} — {적용 기법}

최종 검증: tsc ✅  eslint ✅  vitest {N}개 통과 ✅
```
```
