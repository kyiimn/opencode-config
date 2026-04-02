---
name: init-project-typescript
description: >
  opencode + oh-my-opencode 환경 전용 스킬입니다.
  이 스킬의 기본 동작은 현재 프로젝트(신규·기존·모노레포 서브프로젝트 모두)에
  AGENTS.md와 RULES.md를 생성하는 것입니다.
  별다른 요청이 없어도 스킬이 호출되면 즉시 Step 1(스코프 확인)부터 시작합니다.
  AGENTS.md / RULES.md 생성 외에도 /start-work TDD/SDV 워크플로우 설정
  (Sisyphus 오케스트레이터 + Explore/Librarian 병렬 탐색 + Oracle 아키텍처 자문),
  그리고 그린필드 프로젝트 초기화(사용자 승인 후)까지 한 번에 처리합니다.
  "프로젝트 시작", "agents.md 만들어줘", "opencode 설정", "프로젝트 규칙 파일 생성",
  "AGENTS.md 생성", "코딩 컨벤션 문서화", "프로젝트 초기화", "TDD 설정", "SDV 설정",
  "oh-my-opencode 워크플로" 등의 요청에 반드시 이 스킬을 사용하세요.
---

# init-project-typescript

> **기본 동작**: 이 스킬이 호출되면 별도 지시 없이 **즉시 Step 1(스코프 확인)부터 시작**하여
> 현재 프로젝트에 `AGENTS.md`와 `RULES.md`를 생성한다.
> 신규 프로젝트, 기존 프로젝트, 모노레포 서브프로젝트 모두 동일하다.

opencode + oh-my-opencode 환경에서 **AGENTS.md / RULES.md 생성 → /start-work TDD/SDV 워크플로우 수립 → 그린필드 초기화**를
한 흐름으로 처리하는 스킬입니다. **세 가지 모드**를 지원합니다.

| 모드 | 흐름 |
|------|------|
| **모노레포 루트** | Step 1 → Step 1.5A(소스 있음: 스캔→인터뷰 / 소스 없음: 인터뷰만) → Step 3 → **Step 5(그린필드 시)** |
| **싱글 프로젝트 루트** | Step 1 → Step 1.5B(스캔) → Step 2A(인터뷰) → Step 3 → **Step 5(그린필드 시)** |
| **패키지/앱 단위** | Step 1 → Step 1.5C(스캔) → Step 2B(인터뷰) → Step 3 → Step 4 → **Step 5(그린필드 시)** |

> **그린필드 판별**: AGENTS.md 저장(및 패키지 스코프의 경우 Step 4) 완료 후,
> 해당 경로에 소스코드가 없으면 자동으로 Step 5(프로젝트 초기화 승인 게이트)로 진입한다.

## 이 스킬이 설정하는 것

### AGENTS.md / RULES.md
`AGENTS.md`는 opencode가 모든 세션에서 자동으로 LLM 컨텍스트에 주입하는 프로젝트 규칙 파일입니다.
oh-my-opencode의 Sisyphus 오케스트레이터와 전문 서브에이전트들이 이 파일을 읽고 프로젝트 컨벤션을
준수하며 작업합니다.

### /start-work 워크플로우 (TDD + SDV)

생성되는 AGENTS.md에는 `/start-work` 명령의 에이전트 역할 및 위임 원칙이 명시됩니다.
실제 워크플로우 절차는 `.opencode/commands/start-work.md`에 정의됩니다.

| 에이전트 | 역할 |
|----------|------|
| **Sisyphus** | 오케스트레이터. 계획·위임·워크플로우 강제. 직접 구현 금지 |
| **Explore** | 코드베이스 탐색 (백그라운드 병렬) |
| **Librarian** | 웹/공식문서/OSS 탐색 (백그라운드 병렬) |
| **Oracle** | 아키텍처 자문 (복잡 작업만, 동기 실행) |
| **category:deep** | 복잡 구현·리팩터링·아키텍처 점검 |
| **category:quick** | 단순 수정·lint 정리 |
| **category:visual-engineering** | UI/UX·컴포넌트 구현 |

> **7단계 흐름**: PHASE 0(Prometheus 계획) → PHASE 1(병렬 리서치) → PHASE 2(시나리오+테스트파일) → PHASE 3(승인) → PHASE 4(구현+JSDoc) → PHASE 5(GC) → PHASE 6(검증)

### 파일 구조

```
.opencode/
  oh-my-opencode.json        # 에이전트 모델 설정
  commands/
    start-work.md            # /start-work 워크플로우 정의 (PHASE 1~6)
  test-scenarios/
    {task-name}.md           # 태스크별 TDD/SDV 시나리오 파일
AGENTS.md                    # 에이전트 역할 지침 (본 스킬이 생성)
RULES.md                     # 코딩 규칙 (본 스킬이 생성)
```

---

## Step 1: 스코프 확인

`ask_user_input_v0` 도구로 아래 3개 옵션 중 하나를 선택받는다.

```
Q. 어떤 AGENTS.md를 생성할까요?
  - 모노레포 루트 AGENTS.md      → Step 1.5A로 이동 (소스코드 유무 자동 판별)
  - 싱글 프로젝트 루트 AGENTS.md → Step 1.5B로 이동
  - 패키지/앱 단위 AGENTS.md     → Step 1.5C로 이동 (모노레포의 apps/* 또는 packages/*)
```

선택 결과에 따라 해당 Step으로 즉시 이동한다.
"모노레포 루트"를 선택한 경우 소스코드 존재 여부는 Step 1.5A에서 자동으로 판별한다.

---

## Step 1.5A: 모노레포 루트 AGENTS.md — 소스코드 유무에 따라 분기

스코프가 **모노레포 루트**인 경우, 먼저 `apps/` 또는 `packages/` 하위에 `.ts` / `.tsx` 파일이 존재하는지 확인한다.

```
소스코드 존재 판별 기준:
apps/ 또는 packages/ 하위에 .ts / .tsx 파일이 1개 이상 존재하면 → 소스코드 있음
해당 파일이 전혀 없으면 → 소스코드 없음 (빈 모노레포)
```

---

### Step 1.5A-SCAN: 소스코드가 있는 경우 — 스캔 후 인터뷰

`apps/`·`packages/` 전체를 대상으로 스캔을 수행한다. 스캔 항목과 추론 방법은 Step 1.5C의 **스캔 항목과 추론 방법**을 동일하게 적용한다.

**모노레포 루트 스캔 경로**

| 스캔 대상 | 경로 |
|-----------|------|
| 디렉터리 구조 | `./` (최대 2 depth) |
| TS 파일 수집 | `apps/**/*.ts`, `apps/**/*.tsx`, `packages/**/*.ts`, `packages/**/*.tsx` |
| 패키지 매니저 | `./pnpm-lock.yaml`, `./yarn.lock`, `./package-lock.json` |
| 기술 스택 | `./package.json`, `apps/*/package.json`, `packages/*/package.json` |

스캔이 끝나면 아래 형식으로 결과를 제시하고 `ask_user_input_v0`으로 확인을 받는다.

```
## 코드베이스 스캔 결과

- 파일명 규칙    : {kebab-case | camelCase | PascalCase} ({N}개 파일 중 {M}개 해당)
- 함수명 규칙    : {camelCase | 동사 prefix 강제} ({근거})
- 인터페이스 명명 : {PascalCase | I prefix}
- 패키지 매니저  : {pnpm | yarn | npm}
- 디렉터리 구조  :
  {실제 트리 요약}
```

`ask_user_input_v0` 승인 게이트:
```
Q. 위 스캔 결과가 맞나요?
  - 맞습니다 — 이대로 진행합니다
  - 수정이 필요합니다 — 잘못된 항목을 알려주세요
```

사용자가 "맞습니다"를 선택하거나 수정 내용을 입력하면 그 값을 최종 규칙으로 확정한다.
**승인 또는 수정 확정 전에 다음 단계로 진행하는 것을 금지한다.**
**스캔으로 확정된 항목(Q-N1·N2·N3·N4)은 이후 인터뷰에서 다시 묻지 않는다.**

스캔 확인 후, 스캔으로 확정되지 않은 항목만 `ask_user_input_v0`으로 수집한다.

```
[TypeScript 스타일 정책 — 스캔으로 확정 불가, 항상 질문]

Q-T1. 타입 정의 방식
  - interface 우선  (객체 형태는 interface, 유니온·교차는 type)
  - type 우선       (모든 타입 정의에 type alias 사용)
  - 혼용            (interface와 type을 상황에 따라 선택)

Q-T2. 함수·메서드 선언 방식
  - 화살표 함수 우선   (const fn = () => {})
  - function 선언 우선 (function fn() {})
  - 혼용: 모듈 최상위는 function, 클래스 메서드·콜백은 화살표 함수

```

> Q-N1~N4 중 스캔으로 확정되지 않은 항목이 있으면 해당 질문만 위 항목들과 함께 수집한다.

수집 완료 후:

- 스택·기술 스택 등 나머지 항목은 기본값(`{placeholder}`)으로 채운다
- 모노레포 구조 섹션(`[A:ROOT] STRUCTURE (모노레포 pnpm)`)을 사용한다
- 수집·확정된 네이밍·TS 스타일 값을 `RULES.md#code-standards`의 해당 항목에 반영한다
- 생성 후 사용자에게 "각 앱/패키지에 AGENTS.md를 추가할 때 패키지 스코프로 다시 실행하세요"라고 안내한다
- 바로 Step 3으로 이동

---

### Step 1.5A-EMPTY: 소스코드가 없는 경우 — 인터뷰만 진행 (스캔 생략)

빈 모노레포이므로 스캔을 건너뛰고, **네이밍 규칙(Q-N1~N4)과 TypeScript 스타일 정책(Q-T1~T2)** 6개 항목을 `ask_user_input_v0`으로 수집한다.

```
[네이밍 규칙]

Q-N1. 파일 네이밍 규칙
  - kebab-case  (예: user-service.ts)
  - camelCase   (예: userService.ts)
  - PascalCase  (예: UserService.ts)
  - snake_case  (예: user_service.ts)

Q-N2. 클래스 네이밍 규칙
  - PascalCase  (예: UserService, AuthController)
  - camelCase

Q-N3. 인터페이스 네이밍 규칙
  - PascalCase prefix 없음  (예: User, UserRepository)
  - I prefix + PascalCase   (예: IUser, IUserRepository)

Q-N4. 메서드·함수 네이밍 규칙
  - camelCase + 동사 prefix 강제  (예: getUser, createOrder, handleError)
  - camelCase (동사 prefix 권장이나 강제 아님)

[TypeScript 스타일 정책]

Q-T1. 타입 정의 방식
  - interface 우선  (객체 형태는 interface, 유니온·교차는 type)
  - type 우선       (모든 타입 정의에 type alias 사용)
  - 혼용            (interface와 type을 상황에 따라 선택)

Q-T2. 함수·메서드 선언 방식
  - 화살표 함수 우선   (const fn = () => {})
  - function 선언 우선 (function fn() {})
  - 혼용: 모듈 최상위는 function, 클래스 메서드·콜백은 화살표 함수

```

6개 항목을 수집한 뒤:

- 스택·패키지 매니저·기술 스택 등 나머지 항목은 기본값(`{placeholder}`)으로 채운다
- 모노레포 구조 섹션(`[A:ROOT] STRUCTURE (모노레포 pnpm)`)을 사용한다
- 수집한 네이밍·TS 스타일 값을 `RULES.md#code-standards`의 해당 항목에 반영한다
- 생성 후 사용자에게 "각 앱/패키지에 AGENTS.md를 추가할 때 패키지 스코프로 다시 실행하세요"라고 안내한다
- 바로 Step 3으로 이동

---

## Step 1.5B: 싱글 프로젝트 루트 AGENTS.md — 스캔 후 인터뷰

스코프가 **싱글 프로젝트 루트**인 경우, 코드베이스 스캔과 인터뷰를 통해 AGENTS.md를 생성한다.

### 스캔 대상 경로 (싱글 루트)

`./` 전체 (최대 2 depth), `./src/`

스캔 항목과 추론 방법은 Step 1.5C의 **스캔 항목과 추론 방법**을 동일하게 적용한다.

스캔이 끝나면 Step 1.5C의 **스캔 결과 정리** 형식으로 결과를 제시하고
`ask_user_input_v0` 승인 게이트를 실행한다.

```
Q. 위 스캔 결과가 맞나요?
  - 맞습니다 — 이대로 진행합니다
  - 수정이 필요합니다 — 잘못된 항목을 알려주세요
```

**승인 또는 수정 확정 전에 다음 단계로 진행하는 것을 금지한다.**
스캔으로 확정된 항목은 Step 2A 인터뷰에서 다시 묻지 않는다.
스캔 확인 후 Step 2A(보완 인터뷰)로 이동한다.

---

## Step 1.5C: 코드베이스 스캔 (패키지 스코프 전용, 인터뷰 전 필수)

스코프가 **패키지**인 경우에만 코드베이스 스캔을 수행한다.
스캔으로 파악된 항목은 사용자에게 묻지 않고 그대로 사용한다.

### 스캔 대상 경로 (Step 1.5C 패키지 스코프)

| 스코프 | 스캔 경로 |
|--------|-----------| 
| 패키지 | 사용자가 지정한 패키지 경로 |

> 싱글 프로젝트 루트(Step 1.5B)의 스캔 경로는 `./` 및 `./src/`이며, 해당 항목은 Step 1.5B에 정의되어 있다.

### 스캔 항목과 추론 방법

**1. 파일명 규칙**
실제 `.ts` / `.tsx` 파일명 10개 이상을 수집하여 패턴을 분류한다.
```
kebab-case : user-service.ts, auth-controller.ts
camelCase  : userService.ts, authController.ts
PascalCase : UserService.ts, AuthController.ts
snake_case : user_service.ts, auth_controller.ts
```
파일명이 혼재하면 가장 많이 쓰인 패턴을 채택하고 혼재 사실을 AGENTS.md에 명시한다.

**2. 클래스 네이밍 규칙 (추가)**
`class` 선언을 추출하여 클래스명의 패턴(PascalCase 또는 camelCase)을 판별한다. 대부분의 클래스가 대문자로 시작하면 `PascalCase`로 확정한다.

**3. 함수·메서드 명명**
`*.ts` 파일 3개 이상을 열어 `function`, `const`, 화살표 함수 선언을 추출한다.
동사 prefix(`get`, `create`, `update`, `delete`, `handle`, `on`) 비율이 70% 이상이면
"동사 prefix 강제" 규칙을 채택한다.

**4. 인터페이스 명명**
`interface` 선언을 추출하여 `I` prefix 사용 여부를 판별한다.

**5. 디렉터리 구조**
`ls -R` 또는 파일 트리 조회로 1~2 depth 구조를 파악한다.
`apps/`, `packages/` 존재 여부로 모노레포 여부를 자동 판별한다.

**6. 기술 스택**
`package.json`의 `dependencies` / `devDependencies`를 읽어 자동 판별한다.
기존 소스코드가 존재하는 경우, 실제 의존성 기준으로 자유롭게 판별한다 (아래 4종 외 허용).
기존 소스코드가 없는 경우(그린필드), 반드시 아래 4종 중 하나로만 확정한다.

```
판별 우선순위 (위에서 아래 순서로 첫 일치 항목을 채택):

  express / hono / fastify 존재
    + prisma 존재 (또는 스캔 후 인터뷰에서 확인)
    → API 서버 (Express.js + DDD + Prisma)

  react 존재 (react@19+ 우선)
    → 프론트엔드 (React 19+)

  commander / yargs / oclif 존재
    → CLI 앱

  dependencies가 없거나 위 패턴 미해당
  + 디렉터리가 packages/* 하위
    → 공유 라이브러리

  기존 소스 있고 위 4종 외 스택 감지
    → 실제 스택 그대로 사용 (인터뷰 제한 없음)
```

> 모노레포에서 `apps/*` 하위 패키지는 API 서버 / 프론트엔드 / CLI 앱 중 하나이고,
> `packages/*` 하위 패키지는 공유 라이브러리로 간주한다.

**7. 패키지 매니저**
`pnpm-lock.yaml` → pnpm / `yarn.lock` → yarn / `package-lock.json` → npm

### 스캔 결과 정리

스캔이 끝나면 아래 형식으로 파악된 내용을 정리한다.
인터뷰 전에 사용자에게 간략히 제시하여 잘못 추론된 항목을 수정할 기회를 준다.

```
## 코드베이스 스캔 결과

- 프로젝트 타입 : {싱글 | 모노레포} ({근거: pnpm-workspace.yaml 존재 등})
- 파일명 규칙   : {kebab-case | camelCase | PascalCase} ({N}개 파일 중 {M}개 해당)
- 클래스명 규칙 : {PascalCase | camelCase} ({근거})
- 함수명 규칙   : {camelCase | 동사 prefix 강제} ({근거})
- 인터페이스 명명: {PascalCase | I prefix}
- 기술 스택     : {API 서버(Express+DDD+Prisma) | 프론트엔드(React 19+) | CLI 앱 | 공유 라이브러리 | {기타 — 기존 소스 감지}} ({package.json 근거})
- 패키지 매니저 : {pnpm | yarn | npm}
- 디렉터리 구조 :
  {실제 트리 요약}
```

`ask_user_input_v0` 승인 게이트:
```
Q. 위 스캔 결과가 맞나요?
  - 맞습니다 — 이대로 진행합니다
  - 수정이 필요합니다 — 잘못된 항목을 알려주세요
```

사용자가 "맞습니다"를 선택하거나 수정 내용을 입력하면 그 값을 최종 규칙으로 확정한다.
**승인 또는 수정 확정 전에 다음 단계로 진행하는 것을 금지한다.**
**확정된 항목은 이후 인터뷰에서 다시 묻지 않는다.**

---

## Step 2A: 싱글 프로젝트 루트 AGENTS.md — 보완 인터뷰

**Step 1.5B(싱글 프로젝트 루트 스캔)를 마친 경우에만** 실행한다.
모노레포 루트(Step 1.5A)는 이 단계를 건너뛴다.

스캔으로 파악되지 않은 항목만 `ask_user_input_v0`으로 추가 수집한다.
스캔으로 이미 확정된 항목(파일명 규칙, 기술 스택 등)은 질문 목록에서 제외한다.

**스캔으로 파악 불가한 항목 (항상 질문)**

아래 항목은 소스코드 스캔만으로 확정하기 어렵거나 팀 정책에 따라 달라지므로 반드시 `ask_user_input_v0`으로 수집한다.
단, 스캔에서 이미 명확히 확정된 항목은 해당 질문을 건너뛴다.

```
[네이밍 규칙 — 스캔으로 미확정 시]

Q-N1. 파일 네이밍 규칙 (스캔 결과와 다를 경우 수정 가능)
  - kebab-case  (예: user-service.ts)
  - camelCase   (예: userService.ts)
  - PascalCase  (예: UserService.ts)
  - snake_case  (예: user_service.ts)

Q-N2. 클래스 네이밍 규칙
  - PascalCase  (예: UserService, AuthController)
  - camelCase

Q-N3. 인터페이스 네이밍 규칙
  - PascalCase prefix 없음  (예: User, UserRepository)
  - I prefix + PascalCase   (예: IUser, IUserRepository)

Q-N4. 메서드·함수 네이밍 규칙
  - camelCase + 동사 prefix 강제  (예: getUser, createOrder, handleError)
  - camelCase (동사 prefix 권장이나 강제 아님)

[TypeScript 스타일 정책]

Q-T1. 타입 정의 방식
  - interface 우선  (객체 형태는 interface, 유니온·교차는 type)
  - type 우선       (모든 타입 정의에 type alias 사용)
  - 혼용            (interface와 type을 상황에 따라 선택)

Q-T2. 함수·메서드 선언 방식
  - 화살표 함수 우선   (const fn = () => {})
  - function 선언 우선 (function fn() {})
  - 혼용: 모듈 최상위는 function, 클래스 메서드·콜백은 화살표 함수

[테스트 코드 작성 방식]

```


다음 항목은 open-ended이므로 텍스트로 별도 요청합니다:

> 1. "프로젝트에 대해 간단히 설명해 주세요. (생략 가능)"
> 2. "에이전트가 자주 참조해야 할 핵심 파일 경로를 알려주세요.
>    예: 라우팅 진입점, API 클라이언트, 상태 관리, 디자인 토큰 등. (생략 가능)"
> 3. "참고하면 좋은 예시 파일(Good example), 따라 하면 안 되는 레거시 파일(Bad example)이 있나요? (생략 가능)"

### 루트 생성 규칙

`references/template-agents.md`의 `[A:ROOT]` / `[A]` 섹션으로 **AGENTS.md**를,
`references/template-rules.md`의 `[R]` 섹션으로 **RULES.md**를 각각 생성한다.

**AGENTS.md 조합**
1. 설명이 있으면 `OVERVIEW` 포함
2. 모노레포면 `STRUCTURE (모노레포)`, 싱글이면 `STRUCTURE (싱글)`
3. 핵심 파일 경로가 있으면 `ARCHITECTURE` 포함
4. `COMMANDS`, `SAFETY & PERMISSIONS`, `AGENT WORKFLOW`, `OBSERVABILITY`,
   `LIVING DOCUMENT`, `GIT`, `FILE REFERENCES` 항상 포함
5. 스택별 상세 규칙은 AGENTS.md에 넣지 않는다 — RULES.md에만 기술
6. **토큰 효율 원칙**: AGENTS.md 전체 500토큰(약 400단어) 이내 목표

**RULES.md 조합**
1. `HEADER` 항상 포함 (섹션 목차 포함)
2. `CODE STANDARDS`, `JSDOC`, `TESTING`, `REFACTORING` 항상 포함
3. 기술 스택에 따라 조건부 포함:
   - API 서버 (Express.js + DDD + Prisma) → `API {#api}`
   - 프론트엔드 (React 19+) → `FRONTEND {#frontend}`
   - CLI 앱 → `CLI {#cli}`
   - 공유 라이브러리 → `SHARED {#shared}`
   - 기존 소스 기반 기타 스택 → 해당 스택과 가장 가까운 섹션 포함

**저장 경로**
- `./AGENTS.md` — 현재 작업 디렉터리 루트
- `./RULES.md` — 현재 작업 디렉터리 루트

---

## Step 2B: 패키지 AGENTS.md — 보완 인터뷰

패키지 스코프에서는 **패키지 경로를 반드시 입력받는다.** 경로가 확정되지 않으면 다음 단계로 진행하지 않는다.

```
Q1. 패키지 경로를 입력해 주세요. (필수)
    예: apps/api, apps/web, packages/core
```

경로가 확정된 후 해당 경로를 스캔하여 패키지 종류·기술 스택을 자동 판별한다.
스캔으로 파악되지 않은 항목만 `ask_user_input_v0`으로 추가 수집한다.

**패키지 경로 자동 제안 규칙** (경로를 사용자가 직접 입력하기 전에 제안한다):

```
모노레포 환경에서 패키지 종류 → 권장 경로:
  API 서버      → apps/{서비스명}-api     (예: apps/order-api)
  프론트엔드    → apps/{서비스명}-web     (예: apps/admin-web)
  CLI 앱        → apps/{명령어명}-cli     (예: apps/deploy-cli)
  공유 라이브러리 → packages/{라이브러리명} (예: packages/core)
```

```
Q2. 패키지 종류 (스캔으로 미확정 시에만)
    ※ 그린필드(신규)일 경우 아래 4종 중 하나를 반드시 선택한다.
    ※ 기존 소스가 있는 경우 다른 스택도 허용된다(스캔 결과 우선).

  - API 서버          — Express.js + DDD 아키텍처 + Prisma  (모노레포: apps/*)
  - 프론트엔드        — React 19+                            (모노레포: apps/*)
  - CLI 앱            — commander / yargs 기반               (모노레포: apps/*)
  - 공유 라이브러리   — 타입·유틸리티 공유 패키지             (모노레포: packages/*)
```

**네이밍 규칙·TypeScript 스타일 질문 (루트 RULES.md가 없을 때만)**

루트 `RULES.md`가 이미 존재하면 해당 파일의 `#code-standards`를 읽어 규칙을 상속하며, 아래 질문을 건너뛴다.
루트 `RULES.md`가 없는 경우에만 Step 2A의 Q-N1~N4, Q-T1~T2를 동일하게 수집한다.

패키지 설명은 텍스트로 별도 요청합니다:
> "이 패키지가 담당하는 역할을 간단히 설명해 주세요. (생략 가능)"

### 패키지 생성 규칙

`references/template-agents.md`의 `[A:PKG]` 섹션으로 **패키지 AGENTS.md**를 생성한다.
패키지에는 RULES.md를 별도 생성하지 않는다 — 루트 RULES.md를 공유한다.

1. **루트 AGENTS.md·RULES.md의 내용을 패키지 파일에 반복 기술하지 않는다** — 패키지 특화 구조만 기술
2. 파일 상단에 `[A:PKG] HEADER` 템플릿을 사용하여 **항목별 상속 출처를 테이블로 명시**한다.
   "루트를 따른다"는 표현만으로 끝내지 않는다 — 무엇을 어느 파일·섹션에서 상속하는지 명확히 기술한다.
   상속 테이블에 포함할 항목: TypeScript 설정, 네이밍 규칙, 타입 정의 방식, 함수 선언 방식,
   JSDoc 규칙, 테스트 작성 방식(SDV 포함), 리팩터링 절차, 번들링·실행 도구,
   자율 실행/승인 필요 목록, Git 컨벤션, 에이전트 역할 분담, LIVING DOCUMENT 규칙.
3. 패키지 종류에 따라 해당 STRUCTURE 섹션만 포함:
   - API 서버 (Express.js + DDD + Prisma) → `STRUCTURE[PKG-API]` (규칙 포인터 `RULES.md#api` 포함)
   - 프론트엔드 (React 19+) → `STRUCTURE[PKG-WEB]` (규칙 포인터 `RULES.md#frontend` 포함)
   - CLI 앱 → `STRUCTURE[PKG-CLI]` (규칙 포인터 `RULES.md#cli` 포함)
   - 공유 라이브러리 → `STRUCTURE[PKG-CORE]` (규칙 포인터 `RULES.md#shared` 포함)
4. `AGENT WORKFLOW`, `OBSERVABILITY`, `LIVING DOCUMENT` 항상 포함
5. 저장 경로: `{패키지 경로}/AGENTS.md`

---

## Step 3: 파일 저장

생성한 파일을 **최종 위치에 바로 저장**한다. 중간 경로를 거치지 않는다.

**루트 스코프**: 아래 파일 모두 저장
- `./AGENTS.md`
- `./RULES.md`
- `./.opencode/commands/start-work.md` ← `references/start-work.md` 복사
- `./.opencode/oh-my-opencode.json` ← `references/oh-my-opencode.json` 복사 (없는 경우만)

**패키지 스코프**: 패키지 AGENTS.md만 저장
- `{패키지 경로}/AGENTS.md`

> `.opencode/oh-my-opencode.json` 이 이미 존재하면 덮어쓰지 않는다.
> `.opencode/commands/start-work.md` 는 항상 최신 버전으로 덮어쓴다.

### 파일 작성 도구 제약

**반드시 `write_file` 도구(또는 해당 환경의 파일 작성 전용 도구)를 사용하여 한 번에 파일을 작성한다.**

아래 방법은 이스케이프 문자 손상, 줄바꿈 누락 등의 문제를 일으킬 수 있으므로 금지한다:
- `echo "..." > AGENTS.md` — 따옴표·특수문자 이스케이프 오류 발생 가능
- `cat << EOF > AGENTS.md` — heredoc 들여쓰기 처리 오류 발생 가능
- 기타 쉘 리다이렉션을 통한 파일 생성 일체

저장 완료 후 저장된 경로를 사용자에게 알립니다.
파일 저장 완료 후 저장된 경로를 출력하여 알린다.

---

## Step 4: 루트 AGENTS.md 계층 구조 업데이트 (패키지 스코프 전용)

패키지 AGENTS.md를 새로 저장했다면 **반드시** 이 단계를 실행한다.
루트 AGENTS.md가 없으면 이 단계를 건너뛴다.

### 4-1. 루트 AGENTS.md 위치 확인

```
패키지 경로에서 상위 디렉터리를 순서대로 탐색하여 AGENTS.md를 찾는다.
예: apps/api/AGENTS.md 생성 시 → ./AGENTS.md 탐색
```

### 4-2. 계층 목록 업데이트

루트 AGENTS.md의 `## AGENT WORKFLOW` 섹션 안
`### AGENTS.md 계층 구조` 블록에 새 패키지 경로를 추가한다.

**추가 형식**
```
- `/{패키지 경로}/AGENTS.md` — {패키지 역할 한 줄 요약}
```

**예시: `apps/api/AGENTS.md` 생성 후**
```diff
 ### AGENTS.md 계층 구조
 - `/AGENTS.md` — 프로젝트 전체 규칙 (이 파일)
 - `/packages/core/AGENTS.md` — core 패키지 전용 규칙
+- `/apps/api/AGENTS.md` — REST API 서버 (DDD 아키텍처)
```

이미 동일 경로가 목록에 있으면 역할 설명만 갱신하고 중복 추가하지 않는다.

### 4-3. opencode.json 업데이트 (존재하는 경우)

`.opencode/opencode.json`의 `instructions` 배열에 새 경로가 없으면 추가한다.

```diff
 {
   "instructions": [
     "AGENTS.md",
+    "{패키지 경로}/AGENTS.md",
     "apps/*/AGENTS.md",
     "packages/*/AGENTS.md"
   ]
 }
```
와일드카드 패턴(`apps/*/AGENTS.md`)이 이미 포함된 경우 개별 경로 추가를 생략한다.

### 4-4. 완료 보고 및 Step 5 진입

```
패키지 AGENTS.md 저장 : {패키지 경로}/AGENTS.md ✅
루트 AGENTS.md 업데이트 : /AGENTS.md 계층 목록에 추가 ✅
opencode.json 업데이트 : {업데이트 완료 | 파일 없음으로 건너뜀 | 와일드카드 포함으로 건너뜀} ✅
```

완료 보고 후 **즉시 Step 5(그린필드 판별)로 이동한다.**
패키지 스코프에서도 그린필드이면 초기화를 진행한다.

---

## Step 5: 그린필드 프로젝트 초기화 ⚠️ 사용자 승인 필수

**모든 스코프(루트 / 패키지)에서 실행한다.**
루트 스코프는 Step 3 직후, 패키지 스코프는 Step 4 직후 이 단계로 진입한다.

### 5-1. 그린필드 판별

스코프에 따라 아래 경로를 확인한다. 소스코드가 이미 존재하면 이 단계를 건너뛰고 완료를 보고한다.

```
스코프별 그린필드 판별 기준:

  싱글 프로젝트 루트
    → ./src/ 디렉터리가 존재하지 않으면 → 그린필드

  모노레포 루트
    → ./apps/ 및 ./packages/ 하위에 .ts / .tsx 파일이 없으면 → 그린필드

  패키지/앱 단위
    → {패키지 경로}/src/ 디렉터리가 존재하지 않으면 → 그린필드
```

### 5-2. 초기화 방식 결정

그린필드로 판별되면, **스코프에 따라 초기화 방식을 아래와 같이 결정한다.**

```
스코프 / 기술 스택별 초기화 방식:

  모노레포 루트 (빈 워크스페이스)
    → 스킬 없이 pnpm 워크스페이스 직접 생성 (고정, 선택지 없음)
    → 공통 패키지(vite, vitest 등) 포함

  패키지/앱 단위 또는 싱글 프로젝트
    기술 스택이 "API 서버 (Express.js + DDD + Prisma)"
      → express-api-starter 스킬 자동 추천
    기술 스택이 "프론트엔드 (React 19+)" / "CLI 앱"
      → 직접 지정 또는 스킬 없이 기본 구조
    기술 스택이 "공유 라이브러리" 또는 미확정
      → 스킬 없이 기본 구조 기본 추천
```

### 5-3. 초기화 승인 게이트 ⚠️ 필수

`ask_user_input_v0`으로 확인한다. 승인 없이 파일·디렉터리를 생성하는 것을 절대 금지한다.

**모노레포 루트인 경우** — 방식이 고정이므로 단일 질문만 한다.

```
Q. AGENTS.md 생성이 완료됐습니다.
   빈 모노레포(그린필드)로 감지됐습니다.
   pnpm 워크스페이스 초기 구조를 생성할까요?
   (vite, vitest 등 공통 패키지 포함, 스킬 없이 직접 생성)
     - 예, 생성합니다
     - 아니요, AGENTS.md / RULES.md만 사용합니다
```

**패키지/앱 단위 또는 싱글 프로젝트인 경우** — 방식 선택을 포함한다.

```
Q1. AGENTS.md 생성이 완료됐습니다.
    빈 프로젝트(그린필드)로 감지됐습니다. 프로젝트 초기화를 진행할까요?
      - 예, 지금 바로 초기화합니다
      - 아니요, AGENTS.md만 사용합니다

Q2. (Q1에서 "예"를 선택한 경우) 초기화 방식을 선택하세요.
    [자동 추천된 항목이 있으면 맨 위에 표시]
      - express-api-starter   — Express.js + DDD 아키텍처 API 서버  ← API 프로젝트 자동 추천
      - 직접 지정             — 사용할 스킬 이름을 알려주세요
      - 스킬 없이 기본 구조  — tsconfig / eslint / vitest만 세팅
```

사용자가 "아니요"를 선택하면 완료를 보고하고 스킬을 종료한다.

### 5-4. 초기화 실행

승인이 확정되면 스코프에 따라 아래 방식으로 실행한다.

---

#### A. 모노레포 루트 — pnpm 워크스페이스 직접 생성 (스킬 호출 없음)

아래 파일을 순서대로 직접 생성한다.

**생성 순서 및 파일 목록**

```
1. package.json                        (워크스페이스 루트)
2. pnpm-workspace.yaml
3. tsconfig.json                       (루트 — packages/core/tsconfig/base.json extends)
4. .npmrc
5. packages/core/package.json
6. packages/core/tsconfig/base.json    (모든 패키지 공통 tsconfig)
7. packages/core/tsconfig/app.json     (Node/Bun 앱용)
8. packages/core/tsconfig/react-lib.json  (React 패키지용)
9. packages/core/eslint/index.js       (공통 ESLint flat config)
10. packages/core/eslint/react.js      (React 패키지용 ESLint 확장)
11. packages/core/eslint/node.js       (Node/API 패키지용 ESLint 확장)
12. packages/core/vitest.config.ts     (공통 vitest 설정)
13. packages/core/src/index.ts         (공유 타입·유틸리티 진입점 — 빈 파일)
14. packages/core/src/index.test.ts    (smoke test)
15. apps/                              (빈 디렉터리 — .gitkeep)
```

**각 파일 내용 기준**

```jsonc
// package.json (루트)
{
  "name": "{프로젝트명}",
  "private": true,
  "type": "module",
  "scripts": {
    "build":   "pnpm -r build",
    "dev":     "pnpm -r dev",
    "test":    "pnpm -r test",
    "lint":    "pnpm -r lint",
    "typecheck": "pnpm -r typecheck"
  },
  "devDependencies": {
    "typescript":           "^5",
    "vite":                 "^6",
    "vitest":               "^3",
    "@vitest/coverage-v8":  "^3",
    "eslint":               "^9",
    "typescript-eslint":    "^8",
    "@types/node":          "^22"
  }
}
```

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

```jsonc
// packages/core/tsconfig/base.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noEmit": true,
    "skipLibCheck": true,
    "esModuleInterop": true
  }
}
```

```typescript
// packages/core/vitest.config.ts
import { defineConfig } from 'vitest/config';
export default defineConfig({
  test: {
    globals: false,
    coverage: {
      provider: 'v8',
      reporter: ['text', 'lcov'],
      thresholds: { lines: 80, functions: 80, branches: 80 },
    },
  },
});
```

```typescript
// packages/core/src/index.test.ts  (smoke test)
import { describe, it, expect } from 'vitest';
describe('packages/core smoke test', () => {
  it('workspace is initialized', () => {
    expect(true).toBe(true);
  });
});
```

생성 완료 후 **반드시 루트에서 `pnpm install`을 실행**한다.

---

#### B. 패키지/앱 단위 또는 싱글 프로젝트

**`express-api-starter` 선택 시**:
`express-api-starter` 스킬의 SKILL.md를 먼저 읽고, 스킬 지침에 따라 초기화를 진행한다.
초기화 경로: 패키지 스코프 → `{패키지 경로}/` / 싱글 루트 → `./`

**직접 지정 시**: 사용자가 입력한 스킬명의 SKILL.md를 먼저 읽고 지침을 따른다.

**스킬 없이 기본 구조 선택 시**: 아래 최소 구조를 스코프 경로에 생성한다.

```
{경로}/src/index.ts
{경로}/tsconfig.json        (packages/core/tsconfig/base.json 또는 app.json extends)
{경로}/eslint.config.js     (packages/core/eslint/node.js 또는 react.js import)
{경로}/vitest.config.ts     (packages/core/vitest.config.ts 공유 또는 단독)
{경로}/package.json         (type: module, scripts: typecheck/lint/test)
```

### 5-5. 초기화 후 검증 파이프라인 (자동 실행)

초기화 완료 직후 아래 파이프라인을 **자동으로** 실행한다.
검증 없이 완료를 보고하는 것을 금지한다.

**모노레포 루트인 경우** — `packages/core`를 대상으로 검증한다.

```bash
pnpm install                                    # 워크스페이스 의존성 설치
pnpm --filter @{scope}/core tsc --noEmit        # core 타입 검사
pnpm --filter @{scope}/core lint                # core lint
pnpm --filter @{scope}/core test                # packages/core smoke test 실행
```

**패키지/앱 단위 또는 싱글 프로젝트인 경우**

```bash
pnpm install
pnpm tsc --noEmit
pnpm eslint --fix .
pnpm vitest run
```

명령어가 존재하지 않거나 설정 파일이 없어 실행 불가한 경우 즉시 사용자에게 보고한다.

### 5-6. 완료 보고

**모노레포 루트인 경우**

```
AGENTS.md / RULES.md 저장  : ./AGENTS.md, ./RULES.md ✅
pnpm 워크스페이스 생성      : 스킬 없이 직접 생성 ✅
생성된 파일
  루트                      : package.json, pnpm-workspace.yaml, tsconfig.json, .npmrc
  packages/core             : tsconfig/{base,app,react-lib}.json
                              eslint/{index,react,node}.js
                              vitest.config.ts
                              src/index.ts, src/index.test.ts
  apps/                     : 빈 디렉터리 (.gitkeep)
검증 파이프라인 (packages/core)
  pnpm install              : {PASS | FAIL — 에러 내용}
  tsc --noEmit              : {PASS | FAIL — 에러 내용}
  lint                      : {PASS | FAIL — 에러 내용}
  vitest run (smoke test)   : {PASS | FAIL}

다음 단계:
  - 앱 추가 시: 이 스킬을 "패키지/앱 단위" 모드로 재실행
  - oh-my-opencode 기능 구현 시: /start-work {작업 설명}
```

**패키지/앱 단위 또는 싱글 프로젝트인 경우**

```
AGENTS.md 저장              : {경로}/AGENTS.md ✅
프로젝트 초기화             : {express-api-starter | 스킬명 | 기본 구조} ✅
검증 파이프라인
  tsc --noEmit              : {PASS | FAIL — 에러 내용}
  eslint                    : {PASS | FAIL — 에러 내용}
  vitest run                : {PASS | SKIP(테스트 없음) | FAIL}

다음 단계:
  - oh-my-opencode에서 기능 구현 시: /start-work {작업 설명}
  - 추가 패키지 AGENTS.md 필요 시: 이 스킬을 "패키지/앱 단위" 모드로 재실행
```

---

- **소스코드가 이미 있으면 스캔 결과가 최우선** — 파일명·함수명·디렉터리 구조는 기존 코드를 따른다
- **기술 스택 선택 제한** — 그린필드(신규)에서 인터뷰로 기술 스택을 선택할 때는 반드시 아래 4종 중 하나로 제한한다:
  1. API 서버 (Express.js + DDD 아키텍처 + Prisma)
  2. 프론트엔드 (React 19+)
  3. CLI 앱
  4. 공유 라이브러리
  기존 소스코드가 있는 경우에 한해 위 4종 외의 스택도 허용된다.
- **모노레포 경로 규칙** — 인터뷰에서 선택한 패키지 종류에 따라 경로가 결정된다:
  - API 서버 / 프론트엔드 / CLI 앱 → `apps/*`
  - 공유 라이브러리 → `packages/*`
- 스캔으로 확정된 규칙을 인터뷰에서 다시 묻는 것을 금지한다
- 스캔 결과가 혼재(예: kebab-case 60%, camelCase 40%)하면 우세한 쪽을 채택하고 AGENTS.md에 혼재 사실을 명시한다
- 소스코드가 전혀 없는 빈 디렉터리이면 스캔 단계를 건너뛰고 전체 인터뷰를 진행한다
- **그린필드 초기화 후 검증 필수** — `AGENTS.md`(또는 `RULES.md`)만 존재하는 상태에서 프로젝트를 초기화한 경우, 초기화 완료 즉시 `tsc --noEmit` → `eslint --fix .` → `vitest run` 파이프라인을 자동 실행한다. 검증 없이 완료를 보고하는 것을 금지한다. 생성하는 AGENTS.md의 `AGENT WORKFLOW` 섹션에 이 규칙이 반드시 포함되어 있어야 한다.
- 패키지 AGENTS.md는 **루트 규칙을 항목별 상속 테이블로 명시하고 중복 기술하지 않는다** — 간결함이 핵심
- 에이전트가 매번 읽는 파일이므로 불필요한 설명 최소화
- `any` 타입 금지, strict mode 등 전역 규칙은 루트에만 기술
- 패키지별 파일은 해당 패키지의 디렉터리 구조와 고유 규칙에만 집중

## 스킬 포인터 규칙

생성된 AGENTS.md 안의 `SKILL POINTERS` 섹션이 트리거되는 경우,
해당 스킬을 **먼저 로드한 뒤** 작업을 시작한다. 스킬 로드를 생략하는 것을 금지한다.

| 트리거 | 스킬 |
|--------|------|
| AGENTS.md만 존재하는 빈 프로젝트 + Express.js + DDD + API 초기화 요청 | `express-api-starter` |
| DDD 아키텍처 패키지 AGENTS.md 생성·수정, DDD·헥사고날·레이어드 아키텍처 키워드 등장 | `ddd-architecture` |

생성하는 AGENTS.md에 `SKILL POINTERS` 섹션을 포함할지는 `references/template-agents.md`의 `[A:ROOT] SKILL POINTERS` 섹션 기준을 따른다.
