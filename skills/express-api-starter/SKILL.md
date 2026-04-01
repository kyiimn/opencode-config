---
name: express-api-starter
description: |-
  Express.js + TypeScript API 서버를 Prisma ORM과 DDD 아키텍처로 스캐폴딩합니다.
  새 REST API 백엔드 또는 Express 프로젝트를 처음부터 생성할 때 사용합니다.
  사용자가 "새 API 서버", "Express 프로젝트 시작", "scaffold Express API",
  "Express TypeScript 설정", "API 서버 만들어줘" 등을 말할 때 자동으로 로드하세요.
  예시:
  - user: "새 API 서버 만들어줘" → DB 타입·인증·테스트·IP 제어 순서대로 질문 후 스캐폴딩
  - user: "Express TypeScript 프로젝트 시작" → 옵션 수집 후 전체 프로젝트 구조 생성
compatibility: opencode
---

# Express API Server Starter

새로운 Express.js API 서버 프로젝트를 스캐폴딩합니다.

---

## 핵심 기술 스택

| 영역 | 기술 | 필수 |
|------|------|------|
| **Framework** | Express.js 5.x + TypeScript | ✅ |
| **Build Tool** | Vite (ESM 번들링) + vite-node (개발 서버) | ✅ |
| **Validation** | Zod 4.x (요청 검증) | ✅ |
| **Architecture** | DDD layered architecture — `ddd-architecture` 스킬 참조 | ✅ |
| **Database** | Prisma ORM (PostgreSQL 또는 MySQL 선택) | ✅ |
| **Authentication** | JWT 기반 인증 — `auth-module` 스킬 참조 | 선택 |
| **Testing** | Vitest + supertest | ✅ |
| **Access Control** | IP 기반 접근 제어 (ip-range-check) | 선택 |

---

## 프로젝트 초기화 질문 (필수)

> 스캐폴딩을 시작하기 전, `ask_user_input_v0` 도구를 사용해 아래 항목을 **한 번에** 사용자에게 질문하세요.
> 모든 응답을 수집한 뒤 스캐폴딩을 진행합니다.

```
ask_user_input_v0(questions=[
  {
    question: "DB 엔진을 선택해주세요.",
    type: "single_select",
    options: ["PostgreSQL", "MySQL"]
  },
  {
    question: "JWT 기반 인증 모듈을 포함할까요?",
    type: "single_select",
    options: ["포함", "제외"]
  },
  {
    question: "IP 기반 접근 제어 미들웨어를 포함할까요?",
    type: "single_select",
    options: ["포함", "제외"]
  }
])
```

---

## 프로젝트 구조

DDD 아키텍처와 Prisma는 항상 포함됩니다. 선택 항목은 사용자 응답에 따라 결정됩니다.

```
project-name/
├── prisma/                         # Prisma 스키마 및 마이그레이션
│   ├── schema.prisma
│   └── migrations/
├── src/
│   ├── app.ts                      # Express 앱 진입점
│   ├── lib/
│   │   ├── index.ts                # 라이브러리 barrel export
│   │   ├── db.ts                   # Prisma 클라이언트 설정
│   │   └── db-error.ts             # DB 에러 처리 유틸리티
│   ├── common/
│   │   └── infrastructure/
│   │       └── http/
│   │           └── ip-filter.middleware.ts  # [선택] IP 접근 제어
│   ├── modules/
│   │   ├── auth/                   # [선택] 인증 모듈 (auth-module 스킬 참조)
│   │   └── {feature}/              # 기능 모듈 (ddd-architecture 스킬 참조)
│   ├── test/                       # 테스트
│   │   ├── setup.ts
│   │   └── app.ts
│   └── generated/                  # Prisma 생성 파일 (수정 금지)
├── package.json
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
├── vitest.config.ts                # 테스트 설정
├── prisma.config.ts
└── .env
```

---

## DDD 레이어 구조

> **상세 구현은 `ddd-architecture` 스킬을 참조하세요.**

모든 기능 모듈은 다음 계층으로 구성됩니다.

```
modules/{feature}/
├── domain/                         # 도메인 계층 (인터페이스, DTO, 스키마)
│   ├── {feature}.repository.interface.ts
│   └── {feature}.validation.ts
├── application/                    # 애플리케이션 계층 (비즈니스 로직)
│   └── {feature}.service.ts
└── infrastructure/                 # 인프라 계층 (외부 연동)
    ├── persistence/
    │   └── {feature}.repository.ts # Prisma 구현체
    └── http/
        ├── {feature}.controller.ts
        └── {feature}.route.ts
```

**의존성 방향**: `infrastructure` → `application` → `domain`

---

## 인증 모듈 (선택)

> **상세 구현은 `auth-module` 스킬을 참조하세요.**

- JWT 기반 액세스/리프레시 토큰
- DB에 저장된 리프레시 토큰 관리
- `requireAuth` 미들웨어로 보호된 라우트
- `clientId` 기반 멀티 디바이스 지원

---

## 패키지 설정

### package.json (기본 — 항상 포함)

```json
{
  "name": "api-server",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/app.js",
  "packageManager": "pnpm@10.28.2",
  "scripts": {
    "start": "node dist/app.js",
    "build": "vite build",
    "dev": "vite-node --watch src/app.ts",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate dev",
    "db:push": "prisma db push",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "db:push:test": "prisma db push --skip-generate"
  },
  "dependencies": {
    "@prisma/client": "^7.4.1",
    "cors": "^2.8.5",
    "dotenv": "^17.2.3",
    "express": "^5.1.0",
    "zod": "^4.3.6"
  },
  "devDependencies": {
    "@prisma/config": "^7.4.1",
    "@types/cors": "^2.8.19",
    "@types/express": "^5.0.3",
    "@types/supertest": "^7.2.0",
    "prisma": "^7.4.1",
    "supertest": "^7.2.2",
    "typescript": "^5.8.3",
    "vite": "^6.0.0",
    "vite-node": "^6.0.0",
    "vitest": "^3.2.4"
  }
}
```

### package.json (DB 엔진별 추가 의존성)

**PostgreSQL 선택 시:**
```json
{
  "dependencies": {
    "@prisma/adapter-pg": "^7.4.2",
    "pg": "^8.19.0"
  },
  "devDependencies": {
    "@types/pg": "^8.16.0"
  }
}
```

**MySQL 선택 시:**
```json
{
  "dependencies": {
    "mysql2": "^3.14.0"
  }
}
```

### package.json (선택 항목별 추가 의존성)

```json
{
  "dependencies": {
    "bcrypt": "^6.0.0",            // [선택] 인증 — 비밀번호 해싱
    "ip-range-check": "^0.2.0",    // [선택] IP 기반 접근 제어
    "jose": "^6.1.0"               // [선택] 인증 — JWT
  },
  "devDependencies": {
    "@types/bcrypt": "^6.0.0"      // [선택] 인증
  }
}
```

---

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "es2022",
    "module": "esnext",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "rootDir": "./src",
    "outDir": "./dist",
    "resolveJsonModule": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

> Node.js 백엔드이므로 `"jsx"` 옵션은 포함하지 않습니다.

---

### tsconfig.node.json

```json
{
  "compilerOptions": {
    "composite": true,
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "allowSyntheticDefaultImports": true,
    "strict": true
  },
  "include": ["vite.config.ts", "vitest.config.ts", "prisma.config.ts"]
}
```

---

### vite.config.ts

```typescript
import { defineConfig } from 'vite';
import path from 'path';
import { fileURLToPath } from 'url';
import { builtinModules } from 'module';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

const prodDeps = ['express', 'cors', 'dotenv', 'jose', 'ip-range-check', 'zod'];

// PostgreSQL 사용 시: '@prisma/adapter-pg', 'pg', 'pg-native' 포함
// MySQL 사용 시: 'mysql2' 포함
const nativeModules = ['bcrypt', 'pg', 'pg-native', 'mysql2', '@prisma/adapter-pg', 'prisma'];

const isExternal = (id: string): boolean => {
  if (builtinModules.includes(id) || id.startsWith('node:')) return true;
  if (id.includes('@prisma/client') || id.startsWith('@prisma/') || id.includes('/prisma/')) return true;
  if (nativeModules.some(m => id === m || id.startsWith(`${m}/`))) return true;
  if (prodDeps.includes(id)) return true;
  return false;
};

export default defineConfig({
  build: {
    outDir: 'dist',
    minify: false,
    sourcemap: true,
    ssr: true,
    ssrEmitAssets: false,
    target: 'node22',
    rollupOptions: {
      input: { app: path.resolve(__dirname, 'src/app.ts') },
      external: (id) => isExternal(id),
      output: {
        entryFileNames: '[name].js',
        chunkFileNames: 'chunks/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash][extname]',
        format: 'es',
      },
    },
  },
  resolve: {
    alias: { '@': path.resolve(__dirname, 'src') },
  },
});
```

---

### vitest.config.ts

```typescript
import { defineConfig } from 'vitest/config';
import path from 'path';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    include: ['src/**/*.test.ts', 'src/**/*.spec.ts'],
    testTimeout: 30000,
    hookTimeout: 30000,
    setupFiles: ['./src/test/setup.ts'],
    fileParallelism: false,
    sequence: {
      concurrent: false,
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
});
```

---

### prisma.config.ts

```typescript
import "dotenv/config";
import { defineConfig } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
  },
  datasource: {
    url: process.env["DATABASE_URL"],
  },
});
```

---

## Prisma 스키마

### prisma/schema.prisma — PostgreSQL

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  loginId   String   @unique @db.VarChar(128)
  password  String   @db.VarChar(255)
  name      String   @db.VarChar(128)
  email     String?  @db.VarChar(255)
  isUse     Boolean  @default(true)
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("user")
}

// 인증 모듈이 필요한 경우 auth-module 스킬 참조하여 UserToken 모델 추가
```

### prisma/schema.prisma — MySQL

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid()) @db.VarChar(36)
  loginId   String   @unique @db.VarChar(128)
  password  String   @db.VarChar(255)
  name      String   @db.VarChar(128)
  email     String?  @db.VarChar(255)
  isUse     Boolean  @default(true)
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("user")
}

// 인증 모듈이 필요한 경우 auth-module 스킬 참조하여 UserToken 모델 추가
```

> MySQL에서 `@id @default(uuid())`는 반드시 `@db.VarChar(36)`을 명시해야 합니다.

---

## 핵심 파일 템플릿

### src/app.ts

```typescript
import express, { Application, Request, Response, NextFunction } from "express";
import cors from "cors";

// [선택] IP 기반 접근 제어 포함 시
// import { allowIP } from "@/common/infrastructure/http/ip-filter.middleware";

// [선택] 인증 모듈 포함 시 — auth-module 스킬 참조
// import authRouter from "@/modules/auth/infrastructure/http/auth.route";

const app: Application = express();
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// [선택] IP 접근 제어가 필요한 라우트 예시
// const ADMIN_ALLOWED_IPS = ["::1", "127.0.0.1", "10.10.10.0/24"];
// app.use("/auth", allowIP(ADMIN_ALLOWED_IPS), authRouter);

// 헬스체크
app.get("/ping", (_req: Request, res: Response) => {
  res.status(200).send("pong");
});

// 404 핸들러
app.use((_req: Request, res: Response) => {
  res.status(404).json({ ok: false, message: "요청하신 API를 찾을 수 없습니다." });
});

// 에러 핸들러
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  const error = err as { status?: number; message?: string; stack?: string };
  console.error(error.stack);
  res.status(error.status ?? 500).json({
    ok: false,
    message: error.message ?? "서버 내부 오류가 발생했습니다.",
  });
});

const PORT = process.env.PORT ?? 3000;

app.listen(PORT, () => {
  console.log(`🚀 Server is running on http://localhost:${PORT}`);
});

export default app;
```

---

### src/lib/index.ts

```typescript
export * from "./db";
export * from "./db-error";
```

---

### src/lib/db.ts — PostgreSQL

```typescript
import "dotenv/config";
import { PrismaClient } from "@/generated/client";
import { PrismaPg } from "@prisma/adapter-pg";
import pg from "pg";

const connectionString = process.env.DATABASE_URL;
const pool = new pg.Pool({ connectionString });
const adapter = new PrismaPg(pool);

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    adapter,
    log: process.env.NODE_ENV === "development" ? ["error", "warn"] : ["error"],
  });

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}

process.on("beforeExit", async () => { await prisma.$disconnect(); });
process.on("exit", async () => { await prisma.$disconnect(); });
```

### src/lib/db.ts — MySQL

```typescript
import "dotenv/config";
import { PrismaClient } from "@/generated/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === "development" ? ["error", "warn"] : ["error"],
  });

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}

process.on("beforeExit", async () => { await prisma.$disconnect(); });
process.on("exit", async () => { await prisma.$disconnect(); });
```

> MySQL은 Prisma가 `mysql2`를 내장 드라이버로 사용하므로 별도 어댑터가 필요하지 않습니다.

---

### src/lib/db-error.ts

```typescript
import { Prisma } from "@/generated/client";

export function isPrismaError(error: unknown): error is Prisma.PrismaClientKnownRequestError {
  return error instanceof Prisma.PrismaClientKnownRequestError;
}

export function handleControllerError(error: unknown, context: string) {
  console.error(`[${context}] Error:`, error);

  if (isPrismaError(error)) {
    return {
      status: 400,
      response: { ok: false, message: `데이터베이스 오류: ${error.message}` },
    };
  }

  if (error instanceof Error) {
    return {
      status: 400,
      response: { ok: false, message: error.message },
    };
  }

  return {
    status: 500,
    response: { ok: false, message: "알 수 없는 오류가 발생했습니다." },
  };
}
```

---

### src/common/infrastructure/http/ip-filter.middleware.ts (선택)

```typescript
import { NextFunction, Request, Response } from "express";
import ipRangeCheck from "ip-range-check";

export const allowIP = (allowedCidrs: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const clientIp =
      (req.headers["x-forwarded-for"] as string)?.split(",")[0]?.trim() ||
      req.socket.remoteAddress;

    if (!clientIp) {
      res.status(403).json({ ok: false, message: "IP not detected" });
      return;
    }

    if (!ipRangeCheck(clientIp, allowedCidrs)) {
      res.status(403).json({ ok: false, message: "IP not allowed", ip: clientIp });
      return;
    }

    next();
  };
};
```

---

## 테스트 설정

### src/test/setup.ts

```typescript
import { beforeAll, afterAll } from 'vitest';
import { prisma } from '@/lib/db';

beforeAll(async () => {
  await prisma.$connect();
});

afterAll(async () => {
  await prisma.$disconnect();
});

/**
 * 테스트 간 DB 상태 초기화 함수.
 * 외래키 제약조건 순서를 고려하여 자식 테이블부터 삭제합니다.
 */
export async function cleanAllTables(): Promise<void> {
  // 예: await prisma.userToken.deleteMany({});
  // 예: await prisma.user.deleteMany({});
}

export { prisma };
```

### src/test/app.ts

```typescript
import express from 'express';
import cors from 'cors';
// 테스트 대상 라우터 import
// import authRouter from '@/modules/auth/infrastructure/http/auth.route';

const app = express();
app.use(cors());
app.use(express.json());

// 테스트용 라우트 등록
// app.use('/auth', authRouter);

app.get('/ping', (_req, res) => {
  res.status(200).send('pong');
});

export const createTestApp = () => app;
```

---

## 환경 변수

### .env

```env
# 공통
NODE_ENV=development
PORT=3000

# DB — PostgreSQL
DATABASE_URL="postgresql://user:password@localhost:5432/dbname"

# DB — MySQL (PostgreSQL 대신 사용 시)
# DATABASE_URL="mysql://user:password@localhost:3306/dbname"

# 인증 모듈 포함 시
# JWT_ACCESS_SECRET="your-access-secret-key-min-32-chars"
# JWT_REFRESH_SECRET="your-refresh-secret-key-min-32-chars"
# JWT_ACCESS_EXPIRE="3600"
# JWT_REFRESH_EXPIRE="1209600"
# JWT_HASH_SALT="10"
# PASSWORD_HASH_SALT="10"
```

`.env` 파일은 VCS에 커밋하지 않으며, `.env.example`을 별도로 관리합니다.

---

## 명령어

### 기본

```bash
pnpm dev            # 개발 서버 실행 (hot reload)
pnpm build          # 프로덕션 빌드
pnpm start          # 프로덕션 실행
```

### Prisma

```bash
pnpm db:generate                         # 클라이언트 생성 (schema 변경 후 항상 실행)
pnpm db:migrate --name <migration_name>  # 마이그레이션 생성 및 적용 (개발)
pnpm db:push                             # 스키마 직접 동기화 (개발 전용, 마이그레이션 없음)
pnpm db:push:test                        # 테스트 DB 동기화
```

### 테스트

```bash
pnpm test           # 전체 테스트 실행
pnpm test:watch     # 감시 모드
pnpm test:coverage  # 커버리지 포함 실행
```

---

## 새 모듈 추가 워크플로우

> **상세 구현은 `ddd-architecture` 스킬을 사용하세요.**

1. `domain/{module}.repository.interface.ts` — 인터페이스, DTO 정의
2. `domain/{module}.validation.ts` — Zod 스키마 정의
3. `infrastructure/persistence/{module}.repository.ts` — Prisma 구현체
4. `application/{module}.service.ts` — 비즈니스 로직
5. `infrastructure/http/{module}.controller.ts` — HTTP 핸들러
6. `infrastructure/http/{module}.route.ts` — 라우터 및 DI 조립
7. `app.ts`에 라우터 등록

---

## 인증 모듈 추가 워크플로우 (선택)

> **상세 구현은 `auth-module` 스킬을 사용하세요.**

1. Prisma 스키마에 `UserToken` 모델 추가 → `pnpm db:migrate --name add-user-token`
2. `modules/auth/` 디렉토리 생성
3. `auth-module` 스킬의 템플릿으로 파일 생성
4. `app.ts`에 인증 라우터 등록
5. `.env`에 JWT 시크릿 설정

---

## 관련 스킬

| 스킬 | 용도 | 필수 |
|------|------|------|
| `ddd-architecture` | 새 모듈 생성 시 DDD 레이어 구조 적용 | ✅ |
| `auth-module` | JWT 기반 인증 모듈 구현 | 선택 |

---

## 주의사항

1. **ESM Only**: `"type": "module"` 설정 필수. `require()` 사용 금지.
2. **Path Alias**: 모든 내부 import는 `@/*` alias 사용 (`./src/*` 매핑).
3. **Prisma 생성 파일**: `src/generated/` 디렉토리는 수동 수정 금지. `pnpm db:generate`로만 갱신.
4. **환경 변수**: `.env` 파일은 절대 커밋하지 않음. `.gitignore`에 반드시 추가.
5. **테스트 격리**: 테스트 간 DB 충돌 방지를 위해 `fileParallelism: false` 유지.
6. **IP 미들웨어**: 보안이 필요한 라우트에만 `allowIP` 미들웨어 적용. 전체 앱에 적용하지 않음.
7. **MySQL UUID**: MySQL에서 `@id @default(uuid())`는 `@db.VarChar(36)` 타입을 명시해야 함.
