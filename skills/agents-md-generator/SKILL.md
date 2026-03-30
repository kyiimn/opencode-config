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
**루트 프로젝트용**과 **모노레포 패키지/앱별 패키지용** 두 가지 모드를 지원합니다.

## AGENTS.md란?

`AGENTS.md`는 opencode가 모든 세션에서 자동으로 LLM 컨텍스트에 주입하는 프로젝트 규칙 파일입니다.
oh-my-opencode의 Sisyphus 오케스트레이터와 전문 서브에이전트들이 이 파일을 읽고 프로젝트 컨벤션을
준수하며 작업합니다. opencode는 작업 디렉터리 기준으로 가장 가까운 `AGENTS.md`를 우선 적용하므로,
패키지별 `AGENTS.md`로 컨텍스트를 세밀하게 분리할 수 있습니다.

---

## Step 1: 스코프 확인 (1차 질문)

`ask_user_input_v0` 도구로 **먼저 스코프만** 확인합니다.

```
Q. 어떤 AGENTS.md를 생성할까요?
  - 루트(프로젝트 전체) AGENTS.md
  - 패키지/앱 단위 AGENTS.md (모노레포의 apps/* 또는 packages/*)
```

---

## Step 1: 스코프 확인 (1차 질문)

`ask_user_input_v0` 도구로 **먼저 스코프만** 확인합니다.

```
Q. 어떤 AGENTS.md를 생성할까요?
  - 루트(프로젝트 전체) AGENTS.md
  - 패키지/앱 단위 AGENTS.md (모노레포의 apps/* 또는 packages/*)
```

---

## Step 1.5: 코드베이스 스캔 (인터뷰 전 필수)

스코프가 결정되면 **인터뷰를 시작하기 전에** 반드시 대상 디렉터리의 소스코드를 스캔한다.
스캔으로 파악된 항목은 사용자에게 묻지 않고 그대로 사용한다.

### 스캔 대상 경로

| 스코프 | 스캔 경로 |
|--------|-----------|
| 루트 | `./` 전체 (최대 2 depth), `./src/` 또는 `./apps/`, `./packages/` |
| 패키지 | 사용자가 지정한 패키지 경로 |

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

## Step 2A: 루트 AGENTS.md — 보완 인터뷰

스캔으로 파악되지 않은 항목만 `ask_user_input_v0`으로 추가 수집한다.
스캔으로 이미 확정된 항목(파일명 규칙, 기술 스택 등)은 질문 목록에서 제외한다.

**스캔으로 파악 불가한 항목 (항상 질문)**
```
Q. 포함된 기술 스택 중 스캔으로 확인되지 않은 항목이 있다면 추가 선택
   (스캔 결과에 없는 항목만 표시)
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

스캔으로 패키지 종류·경로·기술 스택이 이미 확정된 경우 해당 질문을 건너뛴다.
파악되지 않은 항목만 `ask_user_input_v0`으로 수집합니다.

```
Q1. 패키지 종류 (스캔으로 미확정 시에만)
  - API 서버 앱 (apps/*)
  - Frontend 앱 (apps/*)
  - CLI 앱 (apps/*)
  - 공유 라이브러리 (packages/*)

Q2. 패키지 이름/경로 (스캔으로 미확정 시에만)
  (텍스트 입력 — 예: apps/api, packages/core)
```

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
