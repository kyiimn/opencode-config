---
name: ddd-architecture
description: >
  Express + Prisma + TypeScript 기반의 DDD(도메인 주도 설계) 레이어 아키텍처 패턴을 정의하고 코드를 생성하는 스킬입니다.
  다음 상황에서 반드시 이 스킬을 사용하세요:
  - 새로운 모듈(module), 도메인(domain), 기능(feature)을 DDD 구조로 생성할 때
  - 레이어별 파일(controller, service, repository, route, validation 등) 작성을 요청받을 때
  - "DDD", "레이어 아키텍처", "도메인 레이어", "퍼시스턴스 레이어", "어플리케이션 레이어" 등의 용어가 등장할 때
  - 기존 DDD 구조에 새 엔드포인트, DTO, 엔티티, 스키마를 추가할 때
  - Express 라우터, Prisma 레포지토리, Zod 스키마를 함께 구성하는 작업을 할 때
  사용자가 단순히 "user 모듈 만들어줘", "CRUD API 작성해줘", "레포지토리 추가해줘"라고 해도, 이 DDD 구조가 적용되어야 한다면 이 스킬을 사용하세요.
---

# DDD Layer Architecture Skill

Express + Prisma + TypeScript 기반의 4계층 DDD 아키텍처를 따릅니다.
세부 코드 예제는 `references/` 폴더를 참고하세요.

---

## 레이어 구조 개요

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

### 의존성 방향

```
HTTP Layer → Application Layer → Domain Layer ← Persistence Layer
```

- **Domain Layer**: 다른 레이어에 의존하지 않음. 인터페이스·DTO·엔티티·스키마 정의.
- **Persistence Layer**: Domain 인터페이스를 구현 (Prisma 사용)
- **Application Layer**: Domain 인터페이스에만 의존 (구현체에 직접 의존 금지)
- **HTTP Layer**: Application 서비스에만 의존

---

## 모듈 디렉토리 구조

```
lib/
└── prisma.ts                              # PrismaClient 싱글톤 (앱 전역 공유)

modules/{module}/
├── domain/
│   ├── {module}.repository.interface.ts   # Repository 인터페이스 + DTO + Entity 타입
│   └── {module}.validation.ts             # Zod 검증 스키마
├── application/
│   └── {module}.service.ts                # 비즈니스 로직
└── infrastructure/
    ├── http/
    │   ├── {module}.controller.ts         # 요청 핸들러 (arrow function)
    │   └── {module}.route.ts              # Express 라우터 + DI 조립
    └── persistence/
        └── {module}.repository.ts         # Prisma 구현체
```

---

## 레이어별 작성 규칙 요약

### 1. Domain Layer

**`{module}.repository.interface.ts`**
- Repository 인터페이스는 `I{Module}Repository` 네이밍
- DTO는 `Create{Module}Dto`, `Update{Module}Dto` 등 접미사 사용
- Entity 타입은 Prisma 타입을 그대로 쓰거나 별도 타입 정의

**`{module}.validation.ts`**
- Zod 스키마로 입력값 검증
- 스키마명은 `create{Module}Schema`, `update{Module}Schema` 등
- `z.infer<typeof schema>` 로 DTO 타입 추출 가능

### 2. Persistence Layer

**`{module}.repository.ts`**
- `I{Module}Repository` 인터페이스를 `implements`
- 생성자에서 `PrismaClient` 주입
- Prisma 호출만 담당, 비즈니스 로직 없음

### 3. Application Layer

**`{module}.service.ts`**
- 생성자에서 `I{Module}Repository` 타입으로 주입받음 (구현체 직접 참조 금지)
- 비즈니스 규칙, 유효성 검사 결과 활용, 트랜잭션 조율
- HTTP 관련 코드(req, res) 절대 포함 금지

### 4. HTTP Layer

**`{module}.controller.ts`**
- 모든 핸들러는 **arrow function** 으로 정의 (`this` 바인딩 문제 방지)
- `req`, `res` 처리만 담당, 비즈니스 로직 없음
- Zod 스키마로 요청 본문 파싱 후 서비스에 전달

**`{module}.route.ts`**
- `express.Router()` 사용
- 컨트롤러 인스턴스를 생성하여 라우터에 등록
- 미들웨어 체이닝 여기서 처리

---

## 의존성 주입(DI) 패턴

생성자 주입 방식을 사용합니다:

```
prisma (lib/prisma.ts 싱글톤)
  → Repository (Prisma 구현체)
    → Service (Repository 인터페이스 타입으로 주입)
      → Controller (Service 주입)
        → Router (Controller 인스턴스 등록)
```

Route 파일에서 조립:
```typescript
import prisma from '../../../../lib/prisma'; // 싱글톤 import

const repository = new {Module}Repository(prisma);
const service = new {Module}Service(repository);
const controller = new {Module}Controller(service);
```

> ⚠️ Route 파일에서 `new PrismaClient()` 직접 생성 금지

---

## 세부 코드 예제

각 레이어의 전체 코드 템플릿은 아래 파일을 참고하세요:

| 파일 | 내용 |
|------|------|
| `references/domain.md` | repository.interface.ts + validation.ts 전체 예제 |
| `references/persistence.md` | repository.ts (Prisma 구현체) 전체 예제 |
| `references/application.md` | service.ts 전체 예제 |
| `references/http.md` | controller.ts + route.ts 전체 예제 |

새 모듈을 만들 때는 해당 레이어의 참조 파일을 읽고 패턴을 따라 작성하세요.

---

## 체크리스트 (새 모듈 추가 시)

- [ ] `domain/{module}.repository.interface.ts` — 인터페이스, DTO, Entity 타입 정의
- [ ] `domain/{module}.validation.ts` — Zod 스키마 정의
- [ ] `infrastructure/persistence/{module}.repository.ts` — 인터페이스 구현
- [ ] `application/{module}.service.ts` — 비즈니스 로직 (인터페이스 타입으로 DI)
- [ ] `infrastructure/http/{module}.controller.ts` — arrow function 핸들러
- [ ] `infrastructure/http/{module}.route.ts` — 라우터 및 DI 조립
- [ ] Application 레이어에서 구현체 직접 import 하지 않는지 확인
- [ ] Controller에 비즈니스 로직이 없는지 확인
