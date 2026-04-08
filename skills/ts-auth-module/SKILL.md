---
name: ts-auth-module
description: >
  Skill for opencode + oh-my-opencode environments. Generates a full JWT auth
  module (login, refresh, logout + authenticate middleware) across all DDD layers
  for Express TypeScript projects. Automatically loaded by the init-ts-api command
  when Q-A2 = include. Also use for requests like "add JWT auth module",
  "generate auth module", "login API", or "token refresh".
---

# ts-auth-module

Generates a JWT-based authentication module across all DDD layers for an Express TypeScript project.

## Context Parameters

This skill expects the following context from the caller:

- `TARGET_DIR` — project root path (e.g. `apps/order-api`)
- `Q-A1` — DB engine (PostgreSQL | MySQL | SQLite)

---

## Files to Generate

```
{TARGET_DIR}/src/modules/auth/
├── domain/
│   ├── auth.repository.interface.ts
│   └── auth.validation.ts
├── application/
│   └── auth.service.ts
└── infrastructure/
    ├── persistence/
    │   └── auth.repository.ts
    └── http/
        ├── auth.middleware.ts
        ├── auth.controller.ts
        └── auth.route.ts
```

---

## Step 1: Prisma Schema Patch

Add the `tokens` relation field to the existing `User` model, then append the `UserToken` model.

### Field to add inside the User model

```prisma
  tokens    UserToken[]
```

### UserToken model — PostgreSQL / MySQL

```prisma
model UserToken {
  id           String   @id @default(uuid())
  userId       String   @map("user_id")
  refreshToken String   @unique @map("refresh_token") @db.VarChar(512)
  expiresAt    DateTime @map("expires_at")
  createdAt    DateTime @default(now()) @map("created_at")

  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("user_token")
}
```

> MySQL: add `@db.VarChar(36)` to `@id @default(uuid())`.

### UserToken model — SQLite

SQLite maps all `String` fields to `TEXT` — omit all `@db.*` annotations:

```prisma
model UserToken {
  id           String   @id @default(uuid())
  userId       String   @map("user_id")
  refreshToken String   @unique @map("refresh_token")
  expiresAt    DateTime @map("expires_at")
  createdAt    DateTime @default(now()) @map("created_at")

  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("user_token")
}
```

After editing the schema, run `pnpm db:generate` (or `pnpm db:migrate`).

---

## Step 2: Create Module Files

Create each file below using `write_file`.

### `src/modules/auth/domain/auth.repository.interface.ts`

```typescript
import { User, UserToken } from "@/generated/client";

export type AuthUserEntity = Omit<User, "password">;

export interface IAuthRepository {
  findUserByLoginId(loginId: string): Promise<User | null>;
  findUserById(id: string): Promise<AuthUserEntity | null>;
  saveRefreshToken(userId: string, token: string, expiresAt: Date): Promise<void>;
  findRefreshToken(token: string): Promise<UserToken | null>;
  deleteRefreshToken(token: string): Promise<void>;
  deleteAllRefreshTokens(userId: string): Promise<void>;
}
```

---

### `src/modules/auth/domain/auth.validation.ts`

```typescript
import { z } from "zod";

export const loginSchema = z.object({
  loginId: z.string().min(1),
  password: z.string().min(1),
});

export const refreshSchema = z.object({
  refreshToken: z.string().min(1),
});

export type LoginInput = z.infer<typeof loginSchema>;
export type RefreshInput = z.infer<typeof refreshSchema>;
```

---

### `src/modules/auth/infrastructure/persistence/auth.repository.ts`

```typescript
import { PrismaClient } from "@/generated/client";
import { IAuthRepository, AuthUserEntity } from "../../domain/auth.repository.interface";
import { User, UserToken } from "@/generated/client";

export class AuthRepository implements IAuthRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findUserByLoginId(loginId: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { loginId } });
  }

  async findUserById(id: string): Promise<AuthUserEntity | null> {
    return this.prisma.user.findUnique({
      where: { id },
      omit: { password: true },
    });
  }

  async saveRefreshToken(
    userId: string,
    token: string,
    expiresAt: Date,
  ): Promise<void> {
    await this.prisma.userToken.create({
      data: { userId, refreshToken: token, expiresAt },
    });
  }

  async findRefreshToken(token: string): Promise<UserToken | null> {
    return this.prisma.userToken.findUnique({
      where: { refreshToken: token },
    });
  }

  async deleteRefreshToken(token: string): Promise<void> {
    await this.prisma.userToken.delete({ where: { refreshToken: token } });
  }

  async deleteAllRefreshTokens(userId: string): Promise<void> {
    await this.prisma.userToken.deleteMany({ where: { userId } });
  }
}
```

---

### `src/modules/auth/application/auth.service.ts`

```typescript
import * as bcrypt from "bcrypt";
import { SignJWT, jwtVerify } from "jose";
import { IAuthRepository, AuthUserEntity } from "../domain/auth.repository.interface";
import { LoginInput } from "../domain/auth.validation";

const ACCESS_SECRET = new TextEncoder().encode(
  process.env.JWT_ACCESS_SECRET ?? "change-me",
);
const REFRESH_SECRET = new TextEncoder().encode(
  process.env.JWT_REFRESH_SECRET ?? "change-me",
);
const ACCESS_EXPIRE_SEC = Number(process.env.JWT_ACCESS_EXPIRE ?? 3600);
const REFRESH_EXPIRE_SEC = Number(process.env.JWT_REFRESH_EXPIRE ?? 1209600);

export interface TokenPair {
  accessToken: string;
  refreshToken: string;
}

export class AuthService {
  constructor(private readonly repository: IAuthRepository) {}

  async login(dto: LoginInput): Promise<TokenPair> {
    const user = await this.repository.findUserByLoginId(dto.loginId);
    if (!user) throw new Error("Invalid credentials");

    const valid = await bcrypt.compare(dto.password, user.password);
    if (!valid) throw new Error("Invalid credentials");

    return this.issueTokens(user.id);
  }

  async refresh(token: string): Promise<{ accessToken: string }> {
    const record = await this.repository.findRefreshToken(token);
    if (!record || record.expiresAt < new Date()) {
      throw new Error("Invalid or expired refresh token");
    }

    try {
      await jwtVerify(token, REFRESH_SECRET);
    } catch {
      throw new Error("Invalid refresh token");
    }

    const accessToken = await this.signAccess(record.userId);
    return { accessToken };
  }

  async logout(token: string): Promise<void> {
    await this.repository.deleteRefreshToken(token).catch(() => {
      // Already deleted — ignore
    });
  }

  async verifyAccessToken(token: string): Promise<AuthUserEntity> {
    const { payload } = await jwtVerify(token, ACCESS_SECRET);
    const userId = payload.sub;
    if (!userId) throw new Error("Invalid token payload");

    const user = await this.repository.findUserById(userId);
    if (!user) throw new Error("User not found");
    return user;
  }

  // ── private ──────────────────────────────────────────────────────────

  private async issueTokens(userId: string): Promise<TokenPair> {
    const [accessToken, refreshToken] = await Promise.all([
      this.signAccess(userId),
      this.signRefresh(userId),
    ]);

    const expiresAt = new Date(Date.now() + REFRESH_EXPIRE_SEC * 1000);
    await this.repository.saveRefreshToken(userId, refreshToken, expiresAt);

    return { accessToken, refreshToken };
  }

  private signAccess(userId: string): Promise<string> {
    return new SignJWT({ sub: userId })
      .setProtectedHeader({ alg: "HS256" })
      .setIssuedAt()
      .setExpirationTime(`${ACCESS_EXPIRE_SEC}s`)
      .sign(ACCESS_SECRET);
  }

  private signRefresh(userId: string): Promise<string> {
    return new SignJWT({ sub: userId })
      .setProtectedHeader({ alg: "HS256" })
      .setIssuedAt()
      .setExpirationTime(`${REFRESH_EXPIRE_SEC}s`)
      .sign(REFRESH_SECRET);
  }
}
```

---

### `src/modules/auth/infrastructure/http/auth.middleware.ts`

```typescript
import { Request, Response, NextFunction } from "express";
import { AuthService } from "../../application/auth.service";
import { AuthUserEntity } from "../../domain/auth.repository.interface";

// Extend Express Request type
declare global {
  namespace Express {
    interface Request {
      user?: AuthUserEntity;
    }
  }
}

export const createAuthenticate = (authService: AuthService) => {
  return async (
    req: Request,
    res: Response,
    next: NextFunction,
  ): Promise<void> => {
    const header = req.headers.authorization;
    if (!header?.startsWith("Bearer ")) {
      res.status(401).json({ ok: false, message: "Unauthorized" });
      return;
    }
    try {
      const token = header.slice(7);
      req.user = await authService.verifyAccessToken(token);
      next();
    } catch {
      res.status(401).json({ ok: false, message: "Invalid or expired token" });
    }
  };
};
```

---

### `src/modules/auth/infrastructure/http/auth.controller.ts`

```typescript
import { Request, Response, NextFunction } from "express";
import { AuthService } from "../../application/auth.service";
import { loginSchema, refreshSchema } from "../../domain/auth.validation";

export class AuthController {
  constructor(private readonly service: AuthService) {}

  login = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const dto = loginSchema.parse(req.body);
      const tokens = await this.service.login(dto);
      res.status(200).json({ ok: true, data: tokens });
    } catch (error) {
      next(error);
    }
  };

  refresh = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const { refreshToken } = refreshSchema.parse(req.body);
      const result = await this.service.refresh(refreshToken);
      res.status(200).json({ ok: true, data: result });
    } catch (error) {
      next(error);
    }
  };

  logout = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const { refreshToken } = refreshSchema.parse(req.body);
      await this.service.logout(refreshToken);
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  };

  me = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      if (!req.user) {
        res.status(401).json({ ok: false, message: "Unauthorized" });
        return;
      }
      res.status(200).json({ ok: true, data: req.user });
    } catch (error) {
      next(error);
    }
  };
}
```

---

### `src/modules/auth/infrastructure/http/auth.route.ts`

```typescript
import { Router } from "express";
import { prisma } from "@/lib/db";
import { AuthRepository } from "../persistence/auth.repository";
import { AuthService } from "../../application/auth.service";
import { AuthController } from "./auth.controller";
import { createAuthenticate } from "./auth.middleware";

const repository = new AuthRepository(prisma);
export const authService = new AuthService(repository);   // re-exportable for reuse
const controller = new AuthController(authService);
export const authenticate = createAuthenticate(authService);

const router = Router();

router.post("/login",   controller.login);
router.post("/refresh", controller.refresh);
router.post("/logout",  controller.logout);
router.get("/me",       authenticate, controller.me);

export default router;
```

---

## Step 3: app.ts Integration

Add the following to `src/app.ts`:

```typescript
// Add import:
import authRouter from "@/modules/auth/infrastructure/http/auth.route";

// Register router (after cors and json middleware):
app.use("/auth", authRouter);
```

---

## Step 4: Environment Variables

Add the following to `.env` and `.env.example` if not already present:

```env
JWT_ACCESS_SECRET="min-32-chars-change-in-production"
JWT_REFRESH_SECRET="min-32-chars-change-in-production"
JWT_ACCESS_EXPIRE="3600"
JWT_REFRESH_EXPIRE="1209600"
```

---

## Step 5: package.json Dependencies

Ensure the following packages are present. Add any that are missing:

```jsonc
// dependencies
"bcrypt": "^6.0.0",
"jose": "^6.1.0",

// devDependencies
"@types/bcrypt": "^6.0.0"
```

---

## Completion Report

```
auth-module
  domain      : auth.repository.interface.ts, auth.validation.ts       ✅
  application : auth.service.ts                                         ✅
  persistence : auth.repository.ts                                      ✅
  http        : auth.middleware.ts, auth.controller.ts, auth.route.ts   ✅

  Endpoints
    POST /auth/login     → { accessToken, refreshToken }
    POST /auth/refresh   → { accessToken }
    POST /auth/logout    → 204
    GET  /auth/me        → authenticated user (requires authenticate middleware)

  Next steps
    1. Patch prisma/schema.prisma — add UserToken model + User.tokens relation
    2. Run pnpm db:generate (or db:migrate)
    3. Set JWT_* environment variables in .env
```
