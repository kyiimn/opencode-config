---
name: express-api-starter
description: Bootstrap a new Express.js API server with TypeScript, Vite, Zod validation, Vitest testing, and IP-based access control. Database (Prisma) and authentication are optional. Use this skill when asked to create, scaffold, or start a new API server project, REST API backend, or Express backend project. Also triggers on mentions of "new API server", "scaffold Express API", "setup Express TypeScript", or "API project starter".
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
| **Architecture** | DDD layered architecture — `ddd-architecture` 스킬 참조 | 선택 |
| **Database** | Prisma ORM with PostgreSQL | 선택 |
| **Authentication** | JWT 기반 인증 — `auth-module` 스킬 참조 | 선택 |
| **Testing** | Vitest + supertest | 선택 |
| **Access Control** | IP 기반 접근 제어 (ip-range-check) | 선택 |

---

## 빠른 시작

새 API 서버 프로젝트를 시작할 때:

### 필수 단계

1. **프로젝트 디렉토리 생성** → 아래 구조 참조
2. **패키지 설정** → `package.json`, `tsconfig.json`, `vite.config.ts` 복사
3. **기본 파일 생성** → `src/app.ts`, `src/lib/`, `src/common/`

### 선택 단계

4. **Prisma 설정** → DB가 필요한 경우 `prisma/schema.prisma`, `prisma.config.ts`
5. **인증 모듈** → 인증이 필요한 경우 `auth-module` 스킬 사용
6. **DDD 모듈 추가** → 새 기능 모듈이 필요한 경우 `ddd-architecture` 스킬 사용

---

## 프로젝트 구조

### 최소 구조 (DB, 인증 없음)

```
project-name/
├── src/
│   ├── app.ts                 # Express 앱 진입점
│   ├── lib/
│   │   └── index.ts           # 라이브러리 barrel export
│   └── modules/               # 기능 모듈들
├── package.json
├── tsconfig.json
├── vite.config.ts
└── .env
```

### 전체 구조 (DB + 인증 + 테스트)

```
project-name/
├── prisma/                    # [선택] DB 사용 시
│   ├── schema.prisma          # Prisma 스키마
│   └── migrations/            # 마이그레이션 파일들
├── src/
│   ├── app.ts                 # Express 앱 진입점
│   ├── lib/
│   │   ├── index.ts           # 라이브러리 barrel export
│   │   ├── db.ts              # [선택] Prisma 클라이언트 설정
│   │   └── db-error.ts        # [선택] DB 에러 처리 유틸리티
│   ├── common/
│   │   └── infrastructure/http/
│   │       └── ip-filter.middleware.ts  # [선택] IP 접근 제어
│   ├── modules/
│   │   ├── auth/              # [선택] 인증 모듈 (auth-module 스킬 참조)
│   │   └── {feature}/         # 기능 모듈들 (ddd-architecture 스킬 참조)
│   ├── test/                  # [선택] 테스트
│   │   ├── setup.ts           # 테스트 설정
│   │   └── app.ts             # 테스트용 Express 앱
│   └── generated/             # [선택] Prisma 생성 파일 (수정 금지)
├── package.json
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
├── vitest.config.ts          # [선택] 테스트 설정
├── prisma.config.ts          # [선택] Prisma 설정
└── .env
```

---

## DDD 레이어 구조 (선택)

> **상세 구현은 `ddd-architecture` 스킬을 참조하세요.**

DDD 아키텍처를 적용하는 경우, 각 모듈은 다음 계층으로 구성됩니다:

```
modules/{feature}/
├── domain/                    # 도메인 계층 (인터페이스, DTO, 스키마)
│   ├── {feature}.repository.interface.ts
│   └── {feature}.validation.ts
├── application/               # 애플리케이션 계층 (비즈니스 로직)
│   └── {feature}.service.ts
└── infrastructure/            # 인프라 계층 (외부 연동)
    ├── persistence/
    │   └── {feature}.repository.ts
    └── http/
        ├── {feature}.controller.ts
        └── {feature}.route.ts
```

**의존성 방향**: `infrastructure` → `application` → `domain`

---

## 인증 모듈 (선택)

> **상세 구현은 `auth-module` 스킬을 참조하세요.**

인증이 필요한 경우:
- JWT 기반 액세스/리프레시 토큰
- DB에 저장된 리프레시 토큰 관리
- `requireAuth` 미들웨어로 보호된 라우트
- `clientId` 기반 멀티 디바이스 지원

---

## 패키지 설정

### package.json (필수)

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
    "dev": "vite-node --watch src/app.ts"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^17.2.3",
    "express": "^5.1.0",
    "zod": "^4.3.6"
  },
  "devDependencies": {
    "@types/cors": "^2.8.19",
    "@types/express": "^5.0.3",
    "vite-node": "^6.0.0"
  }
}
```

### package.json (선택 의존성)

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "db:push:test": "prisma db push --skip-generate"
  },
  "dependencies": {
    "@prisma/adapter-pg": "^7.4.2",    // [선택] Prisma + PostgreSQL
    "@prisma/client": "^7.4.1",         // [선택] Prisma ORM
    "bcrypt": "^6.0.0",                 // [선택] 인증 - 비밀번호 해싱
    "ip-range-check": "^0.2.0",         // [선택] IP 기반 접근 제어
    "jose": "^6.1.0",                   // [선택] 인증 - JWT
    "pg": "^8.19.0",                    // [선택] Prisma PostgreSQL 어댑터
    "shared-types": "workspace:*"       // [선택] 모노레포 공유 타입
  },
  "devDependencies": {
    "@prisma/config": "^7.4.1",         // [선택] Prisma 설정
    "@types/bcrypt": "^6.0.0",          // [선택] bcrypt 타입
    "@types/pg": "^8.16.0",             // [선택] pg 타입
    "@types/supertest": "^7.2.0",       // [선택] 테스트
    "prisma": "^7.4.1",                 // [선택] Prisma CLI
    "supertest": "^7.2.2",              // [선택] 테스트
    "vitest": "^4.0.18"                 // [선택] 테스트
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "es2021",
    "module": "esnext",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "rootDir": "./src",
    "jsx": "react",
    "noImplicitAny": false,
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

### vite.config.ts

```typescript
import { defineConfig } from 'vite';
import path from 'path';
import { fileURLToPath } from 'url';
import { builtinModules } from 'module';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

const prodDeps = ['express', 'cors', 'dotenv', 'jose', 'ip-range-check', 'zod'];
const nativeModules = ['bcrypt', 'pg', 'pg-native', '@prisma/adapter-pg', 'prisma'];

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

## 핵심 파일 템플릿

### src/app.ts (필수)

```typescript
import express, { Application, Request, Response, NextFunction } from "express";
import cors from "cors";

// [선택] IP 기반 접근 제어가 필요한 경우
// import { allowIP } from "@/common/infrastructure/http/ip-filter.middleware";

// [선택] 인증 모듈이 필요한 경우 auth-module 스킬 참조
// import authRouter from "@/modules/auth/infrastructure/http/auth.route";

const app: Application = express();
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// [선택] IP 기반 접근 제어가 필요한 라우트
// const ADMIN_ALLOWED_IPS = ["::1", "127.0.0.1", "10.10.10.0/24"];
// app.use("/auth", allowIP(ADMIN_ALLOWED_IPS), authRouter);

// 헬스체크
app.get("/ping", (req: Request, res: Response) => {
  res.status(200).send("pong");
});

// 404 핸들러
app.use((req: Request, res: Response) => {
  res.status(404).json({ ok: false, message: "요청하신 API를 찾을 수 없습니다." });
});

// 에러 핸들러
app.use((err: any, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    ok: false,
    message: err.message || "서버 내부 오류가 발생했습니다.",
  });
});

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`🚀 Server is running on http://localhost:${PORT}`);
});

export default app;
```

### src/lib/index.ts (필수)

```typescript
// [선택] DB 사용 시 export 추가
// export * from "./db";
// export * from "./db-error";
```

### src/lib/db.ts (선택 - Prisma 사용 시)

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

const cleanup = async () => {
  await prisma.$disconnect();
};

process.on("beforeExit", cleanup);
process.on("exit", cleanup);
```

### src/lib/db-error.ts (선택 - Prisma 사용 시)

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
      response: { ok: false, message: `데이터베이스 오류: ${error.message}` }
    };
  }

  if (error instanceof Error) {
    return {
      status: 400,
      response: { ok: false, message: error.message }
    };
  }

  return {
    status: 500,
    response: { ok: false, message: "알 수 없는 오류가 발생했습니다." }
  };
}
```

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
      res.status(403).json({ message: "IP not detected" });
      return;
    }

    const isAllowed = ipRangeCheck(clientIp, allowedCidrs);

    if (!isAllowed) {
      res.status(403).json({
        message: "IP not allowed",
        ip: clientIp,
      });
      return;
    }

    next();
  };
};
```

---

## 테스트 설정 (선택)

### src/test/setup.ts

```typescript
import { beforeAll, afterAll } from 'vitest';
// [선택] Prisma 사용 시
// import { prisma } from '@/lib/db';

beforeAll(async () => {
  // [선택] Prisma 사용 시
  // await prisma.$connect();
});

afterAll(async () => {
  // [선택] Prisma 사용 시
  // await prisma.$disconnect();
});

// [선택] 테스트용 테이블 정리 함수 (Prisma 사용 시)
// 외래키 제약조건 순서 고려하여 작성
export async function cleanAllTables(): Promise<void> {
  // 예: await prisma.userToken.deleteMany({});
  // 예: await prisma.userInfo.deleteMany({});
}
// export { prisma };
```

### src/test/app.ts

```typescript
import express from 'express';
import cors from 'cors';
// 라우터 import...

const app = express();
app.use(cors());
app.use(express.json());

// 테스트용 라우트 등록
// app.use('/auth', authRouter);
// app.use('/users', userRouter);

app.get('/ping', (req, res) => {
  res.status(200).send('pong');
});

export const createTestApp = () => app;
```

---

## Prisma 스키마 템플릿 (선택)

> Prisma를 사용하는 경우에만 필요합니다.

### prisma/schema.prisma

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 기본 User 모델 예시
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

---

## 환경 변수

### .env.example (필수)

```env
NODE_ENV=development
PORT=3000
```

### .env.example (선택 - DB 사용 시)

```env
DATABASE_URL="postgresql://user:password@localhost:5432/dbname"
```

### .env.example (선택 - 인증 사용 시)

> 인증 모듈 설정은 `auth-module` 스킬 참조

```env
JWT_ACCESS_SECRET="your-access-secret-key-min-32-chars"
JWT_REFRESH_SECRET="your-refresh-secret-key-min-32-chars"
JWT_ACCESS_EXPIRE="3600"
JWT_REFRESH_EXPIRE="1209600"
JWT_HASH_SALT="10"
PASSWORD_HASH_SALT="10"
```

---

## 명령어

### 필수

```bash
# 개발 서버 실행 (hot reload)
pnpm dev

# 프로덕션 빌드
pnpm build

# 프로덕션 실행
pnpm start
```

### 선택 (테스트)

```bash
# 테스트 실행
pnpm test

# 테스트 감시 모드
pnpm test:watch
```

### 선택 (Prisma)

```bash
# Prisma 클라이언트 생성
npx prisma generate

# Prisma 마이그레이션
npx prisma migrate dev --name init

# Prisma 스키마 동기화 (개발용)
npx prisma db push
```

---

## 새 모듈 추가 워크플로우 (선택)

> **상세 구현은 `ddd-architecture` 스킬을 사용하세요.**

1. `domain/{module}.repository.interface.ts` — 인터페이스, DTO 정의
2. `domain/{module}.validation.ts` — Zod 스키마 정의 (선택)
3. `infrastructure/persistence/{module}.repository.ts` — Prisma 구현체
4. `application/{module}.service.ts` — 비즈니스 로직
5. `infrastructure/http/{module}.controller.ts` — HTTP 핸들러
6. `infrastructure/http/{module}.route.ts` — 라우터 및 DI 조립
7. `app.ts`에 라우터 등록

---

## 인증 모듈 추가 워크플로우 (선택)

> **상세 구현은 `auth-module` 스킬을 사용하세요.**

1. Prisma 스키마에 `UserToken` 모델 추가
2. `modules/auth/` 디렉토리 생성
3. `auth-module` 스킬의 템플릿으로 파일 생성
4. `app.ts`에 인증 라우터 등록
5. 환경 변수에 JWT 시크릿 설정

---

## 관련 스킬

| 스킬 | 용도 | 필수 |
|------|------|------|
| `ddd-architecture` | 새 모듈 생성 시 DDD 레이어 구조 적용 | 선택 |
| `auth-module` | JWT 기반 인증 모듈 구현 | 선택 |

---

## 주의사항

### 필수

1. **ESM Only**: `type: "module"` 사용, `require()` 금지
2. **환경 변수**: `.env` 파일은 절대 커밋하지 않음
3. **Path Alias**: `@/*` → `./src/*` 매핑 사용

### 선택

4. **Prisma 생성 파일**: `src/generated/` 디렉토리는 수정 금지 (Prisma 사용 시)
5. **IP 미들웨어**: 보안이 필요한 라우트에 `allowIP` 적용
6. **테스트 격리**: 테스트간 DB 충돌 방지를 위해 `fileParallelism: false` 설정