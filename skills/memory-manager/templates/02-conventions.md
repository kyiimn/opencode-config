# 📜 코딩 컨벤션 및 규칙 (Coding Conventions)

> **업데이트 주기:** 팀의 코딩 스타일이 바뀌거나, AI가 반복적으로 실수하는 패턴을 교정해야 할 때 점진적으로 추가합니다.

## 1. 네이밍 규칙 (Naming Conventions)
- **파일 및 폴더명:** - 컴포넌트 파일: `PascalCase.tsx` (예: `UserProfile.tsx`)
  - 유틸리티 및 일반 파일: `kebab-case.ts` (예: `format-date.ts`)
- **변수 및 함수명:** `camelCase` (예: `getUserData`)
- **상수명:** `UPPER_SNAKE_CASE` (예: `MAX_RETRY_COUNT`)
- **인터페이스/타입명:** `PascalCase`로 작성하되, 앞에 'I'나 'T'를 붙이지 않음 (예: `User` (O), `IUser` (X))

## 2. 코딩 스타일 및 패턴 (Coding Style)
- **컴포넌트 선언:** 화살표 함수(`const Component = () => {}`)를 사용하고 `export default`를 파일 하단에 배치합니다.
- **타입 정의:** `any` 타입 사용을 엄격히 금지합니다. 인터페이스(`interface`)보다 타입(`type`) 별칭을 우선적으로 사용하세요.
- **상태 관리:** - 전역 상태는 가급적 지양하고, 서버 상태는 `React Query`를, 단순 클라이언트 UI 상태는 `useState`를 사용합니다.
- **임포트(Import) 순서:** 1. React 및 외부 라이브러리
  2. 절대 경로 내부 모듈 (`@/components/...`)
  3. 상대 경로 내부 모듈 (`./...`)

## 3. 에러 처리 및 로깅 (Error Handling)
- 모든 비동기 API 호출은 `try-catch` 블록으로 감싸거나 Error Boundary를 통해 처리해야 합니다.
- 사용자에게 노출되는 에러 메시지는 친절한 한국어로 작성하세요. (예: "데이터를 불러오는 데 실패했습니다.")

## 4. Git 및 커밋 메시지 규칙
- **형식:** `<type>(<scope>): <subject>` (Conventional Commits 준수)
- **Type 종류:** `feat` (기능), `fix` (버그 수정), `docs` (문서), `refactor` (리팩토링), `chore` (잡일)
- **예시:** `feat(auth): 카카오 로그인 연동 추가`

## ⚠️ AI 에이전트 특별 지시사항
1. 코드를 생성하거나 수정할 때, 기존 파일의 코드 스타일을 최우선으로 존중하세요.
2. 불필요한 주석은 작성하지 마세요. 코드가 스스로 설명(Self-documenting)하도록 변수/함수명을 명확히 지으세요.
3. 패키지를 새로 설치해야 할 경우, 반드시 `01-architecture.md`에 명시된 Package Manager를 사용하세요.
