---
name: agents-md-generator
description: >
  opencode + oh-my-opencode 프로젝트에서 사용할 AGENTS.md 파일을 인터뷰 기반으로 생성합니다.
  사용자에게 프로젝트 구조, TypeScript 코딩 컨벤션, 포함된 기술 스택(CLI/Frontend/API)을 질문한 뒤,
  opencode와 oh-my-opencode(Sisyphus 오케스트레이션 포함)에서 에이전트가 효과적으로 동작할 수 있도록
  최적화된 AGENTS.md를 생성합니다. "agents.md 만들어줘", "opencode 설정", "프로젝트 규칙 파일 생성",
  "AGENTS.md 생성", "코딩 컨벤션 문서화" 등의 요청에 반드시 이 스킬을 사용하세요.
---

# AGENTS.md Generator

opencode + oh-my-opencode에서 사용할 `AGENTS.md`를 인터뷰 방식으로 생성하는 스킬입니다.
**세 가지 모드**를 지원합니다.

| 모드 | 흐름 |
|------|------|
| **모노레포 루트** | Step 1 → Step 1.5A(소스 있음: 스캔→인터뷰 / 소스 없음: 인터뷰만) → Step 3 |
| **싱글 프로젝트 루트** | Step 1 → Step 1.5B(스캔) → Step 2A(인터뷰) → Step 3 |
| **패키지/앱 단위** | Step 1 → Step 1.5C(스캔) → Step 2B(인터뷰) → Step 3 → Step 4 |

## AGENTS.md란?

`AGENTS.md`는 opencode가 모든 세션에서 자동으로 LLM 컨텍스트에 주입하는 프로젝트 규칙 파일입니다.
oh-my-opencode의 Sisyphus 오케스트레이터와 전문 서브에이전트들이 이 파일을 읽고 프로젝트 컨벤션을
준수하며 작업합니다. opencode는 작업 디렉터리 기준으로 가장 가까운 `AGENTS.md`를 우선 적용하므로,
패키지별 `AGENTS.md`로 컨텍스트를 세밀하게 분리할 수 있습니다.

---

## Step 1: 스코프 확인 (1차 질문)

`ask_user_input_v0` 도구로 **두 가지를 동시에** 확인합니다.

```
Q1. 어떤 AGENTS.md를 생성할까요?
  - 루트(프로젝트 전체) AGENTS.md
  - 패키지/앱 단위 AGENTS.md (모노레포의 apps/* 또는 packages/*)

Q2. (Q1에서 "루트"를 선택한 경우만) 프로젝트 유형은 무엇인가요?
  - 모노레포 (pnpm workspaces / apps·packages 구조)
  - 싱글 프로젝트 (단일 패키지)
```

Q1에서 **패키지/앱 단위**를 선택한 경우 Q2는 묻지 않고 바로 Step 1.5C로 이동한다.

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

스캔이 끝나면 아래 형식으로 결과를 제시하고 사용자 확인을 받는다.

```
## 코드베이스 스캔 결과

- 파일명 규칙    : {kebab-case | camelCase | PascalCase} ({N}개 파일 중 {M}개 해당)
- 함수명 규칙    : {camelCase | 동사 prefix 강제} ({근거})
- 인터페이스 명명 : {PascalCase | I prefix}
- 패키지 매니저  : {pnpm | yarn | npm}
- 디렉터리 구조  :
  {실제 트리 요약}

위 내용이 맞으면 바로 진행합니다. 잘못된 항목이 있으면 알려주세요.
```

사용자가 확인 또는 수정하면 그 값을 최종 규칙으로 확정한다.
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

> Q-N1~N4 중 스캔으로 확정되지 않은 항목이 있으면 해당 질문만 Q-T1·T2와 함께 수집한다.

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

스캔으로 파악된 항목은 사용자에게 묻지 않고 그대로 사용한다.
스캔 완료 후 Step 2A(보완 인터뷰)로 이동한다.

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

**2. 함수·메서드 명명**
`*.ts` 파일 3개 이상을 열어 `function`, `const`, 화살표 함수 선언을 추출한다.
동사 prefix(`get`, `create`, `update`, `delete`, `handle`, `on`) 비율이 70% 이상이면
"동사 prefix 강제" 규칙을 채택한다.

**3. 인터페이스 명명**
`interface` 선언을 추출하여 `I` prefix 사용 여부를 판별한다.

**4. 디렉터리 구조**
`ls -R` 또는 파일 트리 조회로 1~2 depth 구조를 파악한다.
`apps/`, `packages/` 존재 여부로 모노레포 여부를 자동 판별한다.

**5. 기술 스택**
`package.json`의 `dependencies` / `devDependencies`를 읽어 자동 판별한다.
```
React 관련 패키지 존재    → Frontend 포함
express/hono/fastify 존재 → API 서버 포함
commander/yargs 존재      → CLI 툴 포함
prisma 존재               → ORM: Prisma
```

**6. 패키지 매니저**
`pnpm-lock.yaml` → pnpm / `yarn.lock` → yarn / `package-lock.json` → npm

### 스캔 결과 정리

스캔이 끝나면 아래 형식으로 파악된 내용을 정리한다.
인터뷰 전에 사용자에게 간략히 제시하여 잘못 추론된 항목을 수정할 기회를 준다.

```
## 코드베이스 스캔 결과

- 프로젝트 타입 : {싱글 | 모노레포} ({근거: pnpm-workspace.yaml 존재 등})
- 파일명 규칙   : {kebab-case | camelCase | PascalCase} ({N}개 파일 중 {M}개 해당)
- 함수명 규칙   : {camelCase | 동사 prefix 강제} ({근거})
- 인터페이스 명명: {PascalCase | I prefix}
- 기술 스택     : {Frontend / API 서버 / CLI} ({package.json 근거})
- 패키지 매니저 : {pnpm | yarn | npm}
- 디렉터리 구조 :
  {실제 트리 요약}

위 내용이 맞으면 바로 진행합니다. 잘못된 항목이 있으면 알려주세요.
```

사용자가 확인 또는 수정하면 그 값을 최종 규칙으로 확정한다.
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
```

다음 항목은 open-ended이므로 텍스트로 별도 요청합니다:

> 1. "프로젝트에 대해 간단히 설명해 주세요. (생략 가능)"
> 2. "에이전트가 자주 참조해야 할 핵심 파일 경로를 알려주세요.
>    예: 라우팅 진입점, API 클라이언트, 상태 관리, 디자인 토큰 등. (생략 가능)"
> 3. "참고하면 좋은 예시 파일(Good example), 따라 하면 안 되는 레거시 파일(Bad example)이 있나요? (생략 가능)"

### 루트 생성 규칙

`references/template.md`의 `[A:ROOT]` / `[A]` 섹션으로 **AGENTS.md**를,
`[R]` 섹션으로 **RULES.md**를 각각 생성한다.

**AGENTS.md 조합**
1. 설명이 있으면 `OVERVIEW` 포함
2. 모노레포면 `STRUCTURE (모노레포)`, 싱글이면 `STRUCTURE (싱글)`
3. 핵심 파일 경로가 있으면 `ARCHITECTURE` 포함
4. `COMMANDS`, `SAFETY & PERMISSIONS`, `AGENT WORKFLOW`, `OBSERVABILITY`,
   `LIVING DOCUMENT`, `GIT`, `FILE REFERENCES`, `ORCHESTRATION` 항상 포함
5. 스택별 상세 규칙은 AGENTS.md에 넣지 않는다 — RULES.md에만 기술
6. **토큰 효율 원칙**: AGENTS.md 전체 500토큰(약 400단어) 이내 목표

**RULES.md 조합**
1. `HEADER` 항상 포함 (섹션 목차 포함)
2. `CODE STANDARDS`, `JSDOC`, `TESTING`, `REFACTORING` 항상 포함
3. 스택에 따라 조건부 포함:
   - API 포함 → `API {#api}`
   - Frontend 포함 → `FRONTEND {#frontend}`
   - CLI 포함 → `CLI {#cli}`
   - 공유 라이브러리 포함 → `SHARED {#shared}`

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

```
Q2. 패키지 종류 (스캔으로 미확정 시에만)
  - API 서버 앱 (apps/*)
  - Frontend 앱 (apps/*)
  - CLI 앱 (apps/*)
  - 공유 라이브러리 (packages/*)
```

**네이밍 규칙·TypeScript 스타일 질문 (루트 RULES.md가 없을 때만)**

루트 `RULES.md`가 이미 존재하면 해당 파일의 `#code-standards`를 읽어 규칙을 상속하며, 아래 질문을 건너뛴다.
루트 `RULES.md`가 없는 경우에만 Step 2A의 Q-N1~N4, Q-T1~T2를 동일하게 수집한다.

패키지 설명은 텍스트로 별도 요청합니다:
> "이 패키지가 담당하는 역할을 간단히 설명해 주세요. (생략 가능)"

### 패키지 생성 규칙

`references/template.md`의 `[A:PKG]` 섹션으로 **패키지 AGENTS.md**를 생성한다.
패키지에는 RULES.md를 별도 생성하지 않는다 — 루트 RULES.md를 공유한다.

1. **루트 AGENTS.md·RULES.md의 내용을 반복하지 않는다** — 패키지 특화 구조만 기술
2. 파일 상단에 루트 상속 명시
3. 패키지 종류에 따라 해당 STRUCTURE 섹션만 포함:
   - API 앱 → `STRUCTURE[PKG-API]` (규칙 포인터 `RULES.md#api` 포함)
   - Frontend 앱 → `STRUCTURE[PKG-WEB]` (규칙 포인터 `RULES.md#frontend` 포함)
   - CLI 앱 → `STRUCTURE[PKG-CLI]` (규칙 포인터 `RULES.md#cli` 포함)
   - 공유 라이브러리 → `STRUCTURE[PKG-CORE]` (규칙 포인터 `RULES.md#shared` 포함)
4. `AGENT WORKFLOW`, `OBSERVABILITY`, `LIVING DOCUMENT` 항상 포함
5. 저장 경로: `{패키지 경로}/AGENTS.md`

---

## Step 3: 파일 저장

생성한 파일을 **최종 위치에 바로 저장**한다. 중간 경로를 거치지 않는다.

**루트 스코프**: 두 파일 모두 저장
- `./AGENTS.md`
- `./RULES.md`

**패키지 스코프**: 패키지 AGENTS.md만 저장
- `{패키지 경로}/AGENTS.md`

### 파일 작성 도구 제약

**반드시 `write_file` 도구(또는 해당 환경의 파일 작성 전용 도구)를 사용하여 한 번에 파일을 작성한다.**

아래 방법은 이스케이프 문자 손상, 줄바꿈 누락 등의 문제를 일으킬 수 있으므로 금지한다:
- `echo "..." > AGENTS.md` — 따옴표·특수문자 이스케이프 오류 발생 가능
- `cat << EOF > AGENTS.md` — heredoc 들여쓰기 처리 오류 발생 가능
- 기타 쉘 리다이렉션을 통한 파일 생성 일체

저장 완료 후 저장된 경로를 사용자에게 알립니다.
`present_files` 같은 Claude.ai 전용 도구는 사용하지 않습니다.

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

루트 AGENTS.md의 `## AGENT ORCHESTRATION` 섹션 안
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

### 4-4. 완료 보고

```
패키지 AGENTS.md 저장 : {패키지 경로}/AGENTS.md ✅
루트 AGENTS.md 업데이트 : /AGENTS.md 계층 목록에 추가 ✅
opencode.json 업데이트 : {업데이트 완료 | 파일 없음으로 건너뜀 | 와일드카드 포함으로 건너뜀} ✅
```

---

## 주의사항

- **소스코드가 이미 있으면 스캔 결과가 최우선** — 파일명·함수명·디렉터리 구조는 기존 코드를 따른다
- 스캔으로 확정된 규칙을 인터뷰에서 다시 묻는 것을 금지한다
- 스캔 결과가 혼재(예: kebab-case 60%, camelCase 40%)하면 우세한 쪽을 채택하고 AGENTS.md에 혼재 사실을 명시한다
- 소스코드가 전혀 없는 빈 디렉터리이면 스캔 단계를 건너뛰고 전체 인터뷰를 진행한다
- 패키지 AGENTS.md는 **루트 규칙을 중복 기술하지 않는다** — 간결함이 핵심
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

생성하는 AGENTS.md에 `SKILL POINTERS` 섹션을 포함할지는 `references/template.md`의 `[A:ROOT] SKILL POINTERS` 섹션 기준을 따른다.

