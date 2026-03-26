# 🏗️ 아키텍처 및 기술 스택 (Architecture & Tech Stack)

> **업데이트 주기:** 새로운 주요 라이브러리가 도입되거나 인프라 구조가 변경될 때만 업데이트합니다.

## 1. 프로젝트 개요
- **프로젝트명:** [프로젝트 이름을 입력하세요]
- **목적:** [프로젝트의 핵심 비즈니스 로직이나 서비스 목표를 1~2줄로 설명하세요]
- **주요 타겟 환경:** [예: 웹 브라우저(Chrome 중심), iOS/Android 앱, Node.js 서버 등]

## 2. 핵심 기술 스택 (Tech Stack)
### Frontend
- **Framework:** [예: Next.js 14 (App Router), React 18]
- **Language:** [예: TypeScript (Strict mode)]
- **Styling:** [예: Tailwind CSS, CSS Modules]
- **State Management:** [예: Zustand, React Query]

### Backend & Database (해당하는 경우)
- **Framework:** [예: NestJS, Express, Spring Boot]
- **Database:** [예: PostgreSQL (Prisma ORM), MongoDB]
- **API Type:** [예: RESTful API, GraphQL]

### Infrastructure & DevOps
- **Hosting/Deployment:** [예: Vercel, AWS EC2, Docker]
- **Package Manager:** [예: pnpm, yarn, npm] (⚠️ **AI 주의:** 패키지 설치 시 반드시 이 매니저를 사용할 것)

## 3. 주요 디렉터리 구조 (High-level Structure)
이 프로젝트의 최상위 폴더 구조와 각 폴더의 역할입니다.
- `src/app/` : 라우팅 및 페이지 컴포넌트
- `src/components/` : 재사용 가능한 UI 컴포넌트
- `src/hooks/` : 커스텀 리액트 훅
- `src/lib/` : 유틸리티 함수 및 외부 라이브러리 설정 (axios 등)
- `docs/memory/` : AI 및 개발자를 위한 프로젝트 컨텍스트 메모리

## 4. 외부 연동 (3rd Party Integrations)
- **Authentication:** [예: NextAuth.js, Supabase Auth]
- **Payment:** [예: Stripe, Toss Payments]
- **Storage:** [예: AWS S3]
