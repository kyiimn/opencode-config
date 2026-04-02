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

이 파일은 `{PACKAGE_PATH}` 패키지에 특화된 내용만 기술한다.
아래 항목은 루트에서 그대로 상속하며 이 파일에서 재정의하지 않는다.

| 상속 항목 | 출처 | 내용 |
|-----------|------|------|
| TypeScript strict 설정 | `/RULES.md#code-standards` | strict mode, any 금지, 명시적 반환 타입 등 |
| 네이밍 규칙 | `/RULES.md#code-standards` | 파일명·클래스·인터페이스·메서드 규칙 |
| 타입 정의 방식 | `/RULES.md#code-standards` | interface 우선 / type 우선 / 혼용 |
| 함수 선언 방식 | `/RULES.md#code-standards` | 화살표 함수 / function 선언 / 혼용 |
| JSDoc 규칙 | `/RULES.md#jsdoc` | 필수 태그, 작성 예시, 금지 패턴 |
| 테스트 작성 방식 | `/RULES.md#testing` | TDD/SDV 워크플로우, /start-work 시나리오 기반 |
| 리팩터링 절차 | `/RULES.md#refactoring` | PREPARE → IDENTIFY → REFACTOR → VERIFY |
| 번들링·실행 도구 | `/RULES.md#code-standards` | Vite 빌드, vite-node 실행, tsc는 타입 검사 전용 |
| 자율 실행 / 승인 필요 목록 | `/AGENTS.md#safety` | 패키지 설치·삭제, git push, 파일 삭제 등 |
| Git 컨벤션 | `/AGENTS.md#git` | 브랜치 명명, Conventional Commits, PR 전 검증 |
| 에이전트 역할 분담 | `/AGENTS.md#workflow` | Sisyphus·Explore·Librarian·Oracle·category:deep/quick 역할 |
| LIVING DOCUMENT 규칙 | `/AGENTS.md#living` | 규칙 추가 트리거·형식 |

**패키지 역할:** {담당 책임 한 줄}
**패키지 타입:** {API 서버 앱 | Frontend 앱 | CLI 앱 | 공유 라이브러리}
**패키지 전용 규칙:** `/RULES.md#{api | frontend | cli | shared}`
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

모든 기능 구현은 `/start-work {작업 설명}` 명령으로 시작한다.
워크플로우 상세 절차는 `.opencode/commands/start-work.md` 에 정의되어 있다.

**Sisyphus는 오케스트레이터다 — 직접 구현하지 말고 전문 에이전트에게 위임하라.**

### 에이전트 역할

| 에이전트 | 역할 | 실행 방식 |
|----------|------|-----------|
| **Sisyphus** | 오케스트레이터 — 계획, 위임, 워크플로우 강제 | 주 에이전트 |
| **Explore** | 코드베이스 탐색 (읽기 전용) | 백그라운드 병렬 |
| **Librarian** | 웹/공식문서/OSS 탐색 (읽기 전용) | 백그라운드 병렬 |
| **Oracle** | 아키텍처 자문 (읽기 전용) | 동기, 복잡 작업만 |
| **category:deep** | 복잡 구현·리팩터링·아키텍처 점검 | 동기, 승인 후 |
| **category:quick** | 단순 수정·lint 정리 | 동기, 승인 후 |
| **category:visual-engineering** | UI/UX·컴포넌트 구현 | 동기, 승인 후 |
| **category:ultrabrain** | 고난도 아키텍처 설계 | 동기, 필요 시 |

### 위임 원칙

1. **Sisyphus는 직접 구현하지 않는다.** `quick` 수정조차 서브에이전트에게 위임하라.
2. **탐색은 항상 병렬 백그라운드로.** `run_in_background=True`가 기본값이다.
3. **session_id를 재사용하라.** 구현 실패 시 새 에이전트를 만들지 말고 동일 session으로 재위임하라.
4. **JSDoc은 구현과 동시에.** 구현 위임 프롬프트에 반드시 JSDoc 작성 지시를 포함하라.

### /start-work 6단계 흐름

```
PHASE 1: 병렬 리서치     — explore(코드베이스) + librarian(문서/OSS) 동시 발사
PHASE 2: 시나리오 작성   — TDD/SDV 시나리오 파일 + 테스트 파일(Red) 생성
PHASE 3: 사용자 승인     — ask_user_input_v0 + Prisma 스키마 변경 시 db push 승인
PHASE 4: 병렬 구현       — delegate_task + JSDoc 동시 작성
PHASE 5: GC              — 리팩터링 → 미사용 코드 제거 → 아키텍처 위반 점검
PHASE 6: 검증            — lint → tsc → npm test → 시나리오 결과 반영
```
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

## [A:ROOT] SKILL POINTERS

```markdown
## SKILL POINTERS

특정 작업을 시작하기 전에 아래 스킬을 반드시 먼저 로드한다.
스킬 파일을 읽지 않고 작업에 착수하는 것을 금지한다.

| 상황 | 사용할 스킬 |
|------|------------|
| AGENTS.md만 존재하는 빈 프로젝트를 **Express.js + DDD + API** 구조로 초기화할 때 | `express-api-starter` 스킬 |
| DDD 아키텍처 기반 패키지의 AGENTS.md를 생성하거나 수정할 때 | `ddd-architecture` 스킬 |

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
