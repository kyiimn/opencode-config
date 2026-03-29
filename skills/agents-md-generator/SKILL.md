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

## Step 2A: 루트 AGENTS.md — 추가 인터뷰

스코프가 **루트**인 경우 아래 질문을 `ask_user_input_v0`으로 **한 번에** 수집합니다.

```
Q1. 프로젝트 타입
  - 싱글 프로젝트
  - 모노레포 (pnpm workspaces)

Q2. 포함된 기술 스택 (복수 선택)
  - CLI 툴
  - Frontend (React 19)
  - API 서버

Q3. 파일명 규칙
  - kebab-case (my-component.ts)
  - camelCase (myComponent.ts)
  - PascalCase (MyComponent.ts)
  - snake_case (my_component.ts)

Q4. 인터페이스 명명
  - PascalCase (UserService)
  - I prefix PascalCase (IUserService)

Q5. 함수/메서드 명명
  - camelCase (getUserById)
  - 동사 prefix 강제 camelCase (getX / createX / updateX / deleteX)
```

다음 항목은 open-ended이므로 텍스트로 별도 요청합니다:

> 1. "프로젝트에 대해 간단히 설명해 주세요. (생략 가능)"
> 2. "에이전트가 자주 참조해야 할 핵심 파일 경로를 알려주세요.
>    예: 라우팅 진입점, API 클라이언트, 상태 관리, 디자인 토큰 등. (생략 가능)"
> 3. "참고하면 좋은 예시 파일(Good example), 따라 하면 안 되는 레거시 파일(Bad example)이 있나요? (생략 가능)"

### 루트 생성 규칙

`references/template.md`의 **[ROOT]** 섹션을 참고하여 조합합니다.

1. 프로젝트 설명이 있으면 `OVERVIEW` 포함
2. 모노레포면 전체 `apps/` + `packages/` 구조, 싱글이면 단일 구조
3. 핵심 파일 경로가 있으면 `ARCHITECTURE` 섹션 포함
4. Good/Bad 파일이 있으면 `CODE EXAMPLES` 섹션 포함
5. 선택된 스택마다 전용 섹션 추가
6. `COMMANDS`, `CODE STANDARDS`, `SAFETY & PERMISSIONS`, `AGENT WORKFLOW`,
   `TESTING`, `GIT`, `FILE REFERENCES`, `ORCHESTRATION` 항상 포함
7. **토큰 효율 원칙**: 생성된 파일 전체가 500토큰(약 400단어) 이내를 목표로 한다.
   상세 규칙은 파일 참조(`FILE REFERENCES`)로 대체하고, AGENTS.md에는 핵심 휴리스틱만 남긴다.
8. 저장 경로: `./AGENTS.md` (현재 작업 디렉터리 루트에 직접 저장)

---

## Step 2B: 패키지 AGENTS.md — 추가 인터뷰

스코프가 **패키지/앱**인 경우 아래 질문을 `ask_user_input_v0`으로 **한 번에** 수집합니다.

```
Q1. 패키지 종류
  - API 서버 앱 (apps/*)
  - Frontend 앱 (apps/*)
  - CLI 앱 (apps/*)
  - 공유 라이브러리 (packages/*)

Q2. 패키지 이름/경로
  (텍스트 입력 — 예: apps/api, packages/core)
```

패키지 설명은 텍스트로 별도 요청합니다:

> "이 패키지가 담당하는 역할을 간단히 설명해 주세요. (생략 가능)"

### 패키지 생성 규칙

`references/template.md`의 **[PACKAGE]** 섹션을 참고하여 조합합니다.

1. **루트 AGENTS.md의 규칙을 반복하지 않는다** — 패키지 특화 내용만 기술
2. 파일 상단에 루트 AGENTS.md 상속 명시
3. 패키지 종류에 따라 해당 섹션만 포함:
   - API 앱 → DDD 구조 + 모듈 패턴 + API 규칙
   - Frontend 앱 → 컴포넌트 구조 + React 규칙
   - CLI 앱 → 커맨드 구조 + CLI 규칙
   - 공유 라이브러리 → export 규칙 + 의존성 규칙
4. 저장 경로: `{패키지 경로}/AGENTS.md` (예: `apps/api/AGENTS.md`)
   현재 작업 디렉터리 기준 상대 경로로 직접 저장한다.

---

## Step 3: 파일 저장

생성한 `AGENTS.md`를 **최종 위치에 바로 저장**합니다. 중간 경로를 거치지 않습니다.
반드시 에디터 환경에 내장된 **파일 작성 전용 도구(예: `write_file`)를 사용하여 한 번에 파일을 작성**하세요.
`echo`나 `cat` 같은 쉘 명령어를 통한 파일 생성은 이스케이프 문자 깨짐 등의 오류가 발생할 수 있으므로 절대 금지합니다.

- **루트 AGENTS.md** → `./AGENTS.md` (현재 작업 디렉터리 루트)
- **패키지 AGENTS.md** → `{패키지 경로}/AGENTS.md` (예: `apps/api/AGENTS.md`)

저장 후 저장된 경로를 사용자에게 알립니다.
`present_files` 같은 Claude.ai 전용 도구는 사용하지 않습니다.

---

## 주의사항

- 패키지 AGENTS.md는 **루트 규칙을 중복 기술하지 않는다** — 간결함이 핵심
- 에이전트가 매번 읽는 파일이므로 불필요한 설명 최소화
- `any` 타입 금지, strict mode 등 전역 규칙은 루트에만 기술
- 패키지별 파일은 해당 패키지의 디렉터리 구조와 고유 규칙에만 집중
