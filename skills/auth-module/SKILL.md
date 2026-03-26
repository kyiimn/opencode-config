---
name: auth-module
description: |
  Implement JWT-based authentication with access/refresh tokens, database token management, and middleware protection. Use this skill when building auth systems, adding login/refresh/logout endpoints, creating protected routes, or implementing token rotation patterns with Express.js and Prisma.
  
  Triggers: "add authentication", "implement login", "JWT auth", "refresh token", "auth middleware", "protected routes", "token rotation", "signin signout", "auth module".
---

# Auth Module Implementation Guide

Complete JWT-based authentication system with access/refresh token pairs, database-managed refresh tokens, and middleware protection. Follows Domain-Driven Design (DDD) architecture for maintainability.

## Quick Start

1. Set environment variables (see Configuration section)
2. Add Prisma schema for `UserToken` model
3. Create the layered module structure: `domain/` → `application/` → `infrastructure/`
4. Wire routes with dependency injection

## Architecture Overview

```
modules/auth/
├── domain/
│   └── auth.repository.interface.ts   # Repository contract
├── application/
│   └── auth.service.ts                # Business logic: JWT, signin, refresh
├── infrastructure/
│   ├── http/
│   │   ├── auth.controller.ts        # HTTP handlers
│   │   ├── auth.route.ts             # Express router + DI wiring
│   │   └── auth.middleware.ts        # requireAuth middleware factory
│   └── persistence/
│       └── auth.repository.ts        # Prisma implementation
└── __tests__/
    └── auth.test.ts                  # Comprehensive test suite
```

**Data Flow:**
```
Request → Middleware (requireAuth) → Controller → Service → Repository → Database
                                    ↓
                              res.locals.user (JWT payload)
```

## Token Patterns

### Access/Refresh Token Pair

- **Access Token**: Short-lived (60s default), stateless JWT
- **Refresh Token**: Long-lived (14 days), stored hashed in database
- **Rotation**: New token pair generated on every refresh
- **Multi-device**: Different `clientId` values for separate sessions

### Token Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│                        SIGNIN FLOW                              │
├─────────────────────────────────────────────────────────────────┤
│  1. Validate credentials (Zod schema)                           │
│  2. Find user by loginId + include tokens                      │
│  3. Verify password (bcrypt compareSync)                        │
│  4. Check active status (isUse !== false)                      │
│  5. Generate token pair (JWT + separate secrets)                │
│  6. Hash refresh token (bcrypt with JWT_HASH_SALT)              │
│  7. Upsert token: (userId, type, clientId) → hashed token       │
│  8. Return { accessToken, refreshToken, user }                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       REFRESH FLOW                              │
├─────────────────────────────────────────────────────────────────┤
│  1. Validate refresh token (Zod schema)                        │
│  2. Verify JWT with JWT_REFRESH_SECRET                          │
│  3. Find user + tokens                                          │
│  4. Compare: provided token vs stored hash (bcrypt)             │
│  5. Generate NEW token pair                                      │
│  6. Hash new refresh token, upsert in database                  │
│  7. Return { accessToken, refreshToken }                        │
└─────────────────────────────────────────────────────────────────┘
```

## Core Implementation

### 1. Prisma Schema

```prisma
model UserToken {
  id        BigInt        @id @default(autoincrement())
  userId    String        @map("user_id")
  clientId  String        @map("client_id") @db.VarChar(255)
  type      UserTokenType @default(REFRESH)
  token     String        @db.VarChar(255)  // Hashed refresh token
  createdAt DateTime      @default(now()) @map("created_at")
  updatedAt DateTime      @updatedAt @map("updated_at")

  user UserInfo @relation(fields: [userId], references: [id])

  @@unique([userId, type, clientId])  // One token per user/client
  @@map("user_token")
}

enum UserTokenType {
  ACCESS
  REFRESH
}
```

**Key Pattern**: Composite unique key `(userId, type, clientId)` enables:
- One refresh token per device/client
- Token replacement on refresh (upsert, not insert)
- Multi-device support

### 2. Domain Interface

`domain/auth.repository.interface.ts`:

```typescript
import { JWTPayload } from "@shared-types/common/jwt-payload";

export interface IAuthRepository {
  findUserForAuthentication(loginId: string, clientId: string): Promise<{
    id: string;
    loginId: string;
    password: string;
    name: string;
    isUse: boolean;
    tokens: { id: bigint; token: string; type: string; clientId: string }[];
  } | null>;

  upsertRefreshToken(
    userId: string,
    clientId: string,
    hashedToken: string
  ): Promise<void>;

  deleteRefreshTokens(userId: string, clientId: string): Promise<void>;
}
```

### 3. AuthService (Application Layer)

Core business logic for token generation, verification, and auth flows.

**Token Generation:**
```typescript
private async _generateToken(payload: JWTPayload) {
  const curTime = new Date().getTime() / 1000;
  const textEncoder = new TextEncoder();
  
  const accessToken = await new SignJWT({ ...payload })
    .setProtectedHeader({ alg: "HS256" })
    .setExpirationTime(parseInt(process.env.JWT_ACCESS_EXPIRE || "60") + curTime)
    .sign(textEncoder.encode(process.env.JWT_ACCESS_SECRET));

  const refreshToken = await new SignJWT({ ...payload })
    .setProtectedHeader({ alg: "HS256" })
    .setExpirationTime(parseInt(process.env.JWT_REFRESH_EXPIRE || "1209600") + curTime)
    .sign(textEncoder.encode(process.env.JWT_REFRESH_SECRET));

  return { accessToken, refreshToken };
}
```

**Token Verification:**
```typescript
public async verifyToken(accessToken: string): Promise<JWTPayload | null> {
  try {
    const { payload } = await jwtVerify<JWTPayload>(
      accessToken,
      new TextEncoder().encode(process.env.JWT_ACCESS_SECRET)
    );
    return payload;
  } catch {
    return null;
  }
}
```

**Important**: The `_generateToken` method is private. Public methods (`signin`, `refresh`, `signout`) call it internally.

### 4. Middleware Factory Pattern

`infrastructure/http/auth.middleware.ts`:

```typescript
export const requireAuth = (authService: AuthService) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    const authHeader = req.header("Authorization");
    const accessToken = authHeader?.split("Bearer ")[1];

    if (!accessToken) {
      res.status(401).json({ ok: false, message: "인증 토큰이 필요합니다." });
      return;
    }

    const payload = await authService.verifyToken(accessToken);
    if (!payload) {
      res.status(401).json({ ok: false, message: "유효하지 않은 토큰입니다." });
      return;
    }

    res.locals.user = payload;  // Attach user to response locals
    next();
  };
};
```

**Usage Pattern:**
```typescript
const authMiddleware = requireAuth(authService);

// Apply to protected routes
router.post("/signout", authMiddleware, authController.signout);
router.get("/me", authMiddleware, (req, res) => {
  const user = res.locals.user;
  res.json({ ok: true, data: user });
});
```

**Why Factory?** Dependency injection allows testing with mock AuthService.

### 5. Repository Implementation

`infrastructure/persistence/auth.repository.ts`:

```typescript
export class AuthRepository implements IAuthRepository {
  constructor(private prisma: PrismaClient) {}

  async findUserForAuthentication(loginId: string, clientId: string) {
    return this.prisma.userInfo.findUnique({
      where: { loginId },
      include: {
        tokens: {
          where: { clientId }
        }
      }
    });
  }

  async upsertRefreshToken(
    userId: string,
    clientId: string,
    hashedToken: string
  ): Promise<void> {
    await this.prisma.userToken.upsert({
      where: {
        userId_type_clientId: { userId, type: "REFRESH", clientId }
      },
      update: { token: hashedToken },
      create: { userId, clientId, type: "REFRESH", token: hashedToken }
    });
  }

  async deleteRefreshTokens(userId: string, clientId: string): Promise<void> {
    await this.prisma.userToken.deleteMany({
      where: { userId, clientId, type: "REFRESH" }
    });
  }
}
```

### 6. Controller Pattern

`infrastructure/http/auth.controller.ts`:

```typescript
export class AuthController {
  constructor(private authService: AuthService) {}

  signin = async (req: Request, res: Response) => {
    try {
      const { loginId, password, clientId } = signinSchema.parse(req.body);
      const result = await this.authService.signin(loginId, password, clientId);
      res.json({ ok: true, data: result });
    } catch (error) {
      handleControllerError(res, error);
    }
  };

  refresh = async (req: Request, res: Response) => {
    try {
      const { refreshToken, clientId } = refreshTokenSchema.parse(req.body);
      const result = await this.authService.refresh(refreshToken, clientId);
      res.json({ ok: true, data: result });
    } catch (error) {
      handleControllerError(res, error);
    }
  };

  signout = async (req: Request, res: Response) => {
    try {
      const { clientId } = signoutSchema.parse(req.body);
      const user = res.locals.user;
      await this.authService.signout(user.id, clientId);
      res.json({ ok: true });
    } catch (error) {
      handleControllerError(res, error);
    }
  };
}
```

**Key Pattern**: Zod validation at controller entry, service handles business logic.

### 7. Route Wiring (Dependency Injection)

`infrastructure/http/auth.route.ts`:

```typescript
import { Router } from "express";
import { AuthRepository } from "../persistence/auth.repository";
import { AuthService } from "../../application/auth.service";
import { AuthController } from "./auth.controller";
import { requireAuth } from "./auth.middleware";

const router = Router();

// Dependency injection
const authRepository = new AuthRepository();
const authService = new AuthService(authRepository);
const authController = new AuthController(authService);
const authMiddleware = requireAuth(authService);

// Public routes
router.post("/signin", authController.signin);
router.post("/refresh", authController.refresh);

// Protected routes
router.post("/signout", authMiddleware, authController.signout);
router.get("/me", authMiddleware, authController.getCurrentUser);

export default router;
```

## Configuration

### Environment Variables

```bash
# .env
JWT_ACCESS_EXPIRE=60           # Access token TTL in seconds (default: 60)
JWT_ACCESS_SECRET=<hex-secret> # HS256 signing key for access tokens
JWT_REFRESH_SECRET=<hex-secret># HS256 signing key for refresh tokens
JWT_REFRESH_EXPIRE=1209600     # Refresh token TTL in seconds (14 days)
JWT_HASH_SALT=10               # bcrypt rounds for refresh token hashing
PASSWORD_HASH_SALT=10          # bcrypt rounds for password hashing
```

**Security Notes:**
- Use different secrets for access and refresh tokens
- Access tokens should be short-lived (60-300 seconds)
- Refresh tokens should be long-lived (7-14 days)
- `JWT_HASH_SALT` controls bcrypt cost factor — balance security vs performance

### JWTPayload Type

`packages/shared-types/src/common/jwt-payload.ts`:

```typescript
export interface JWTPayload {
  id: string;
  loginId: string;
  name: string;
  // Add custom fields as needed
}
```

Share this between frontend and backend for type safety.

### Zod Validation Schemas

`packages/shared-types/src/validation/auth.ts`:

```typescript
import { z } from "zod";

export const signinSchema = z.object({
  loginId: z.string().min(1),
  password: z.string().min(1),
  clientId: z.string().min(1),
});

export const refreshTokenSchema = z.object({
  refreshToken: z.string().min(1),
  clientId: z.string().min(1),
});

export const signoutSchema = z.object({
  clientId: z.string().min(1),
});
```

## Security Considerations

### Token Storage (Client-Side)

| Option | Access Token | Refresh Token |
|--------|--------------|---------------|
| Memory | ✅ Recommended | ✅ Recommended |
| httpOnly Cookie | ⚠️ CSRF risk | ✅ Good option |
| localStorage | ❌ XSS vulnerable | ❌ Never |
| sessionStorage | ⚠️ XSS vulnerable | ⚠️ Less ideal |

### Password Hashing

```typescript
// When creating users
const hashedPassword = await bcrypt.hash(password, parseInt(process.env.PASSWORD_HASH_SALT || "10"));

// When verifying
const isValid = bcrypt.compareSync(password, user.password);
```

### Refresh Token Hashing

Always hash refresh tokens before storing:
```typescript
const hashedRefreshToken = await bcrypt.hash(refreshToken, parseInt(process.env.JWT_HASH_SALT || "10"));
await authRepository.upsertRefreshToken(userId, clientId, hashedRefreshToken);
```

**Why?** If database is compromised, attackers cannot use hashed tokens directly.

## API Reference

| Endpoint | Method | Auth | Body | Response |
|----------|--------|------|------|----------|
| `/auth/signin` | POST | No | `{ loginId, password, clientId }` | `{ ok, data: { accessToken, refreshToken, user } }` |
| `/auth/refresh` | POST | No | `{ refreshToken, clientId }` | `{ ok, data: { accessToken, refreshToken } }` |
| `/auth/signout` | POST | Yes | `{ clientId }` | `{ ok }` |
| `/auth/me` | GET | Yes | - | `{ ok, data: user }` |

**Note**: `clientId` identifies the client device. Generate a UUID per device/session on the frontend.

## Testing Patterns

### Unit Testing AuthService

```typescript
describe("AuthService", () => {
  it("should verify access token successfully", async () => {
    const { accessToken } = await authService.signin("user", "pass", "client1");
    const payload = await authService.verifyToken(accessToken);
    expect(payload).toBeDefined();
    expect(payload.loginId).toBe("user");
  });

  it("should reject invalid refresh token", async () => {
    await expect(
      authService.refresh("invalid-token", "client1")
    ).rejects.toThrow("유효하지 않은 리프레시 토큰입니다.");
  });
});
```

### Integration Testing Routes

```typescript
describe("Auth Routes", () => {
  it("POST /auth/signin should return tokens", async () => {
    const res = await request(app)
      .post("/auth/signin")
      .send({ loginId: "test", password: "test", clientId: "test-client" });
    
    expect(res.status).toBe(200);
    expect(res.body.data.accessToken).toBeDefined();
  });

  it("GET /auth/me should require auth", async () => {
    const res = await request(app).get("/auth/me");
    expect(res.status).toBe(401);
  });
});
```

## Common Patterns

### Multi-Tenant / Multi-Database Support

If using Prisma multi-database pattern:
```typescript
// Inject PrismaClient based on tenant
const authRepository = new AuthRepository(tenantPrismaClient);
```

### Rate Limiting

Add rate limiting to signin endpoint:
```typescript
import rateLimit from "express-rate-limit";

const signinLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per window
  message: { ok: false, message: "Too many login attempts" }
});

router.post("/signin", signinLimiter, authController.signin);
```

### IP-Based Access Control

For admin routes, use IP filtering middleware:
```typescript
import { allowIP } from "../../common/infrastructure/http/ip-filter.middleware";

const ADMIN_ALLOWED_IPS = ["::1", "127.0.0.1", "10.10.10.0/24"];

router.use("/admin", allowIP(ADMIN_ALLOWED_IPS), adminRoutes);
```

## Files Reference

| File | Purpose |
|------|---------|
| `auth.repository.interface.ts` | Repository contract (domain layer) |
| `auth.service.ts` | Business logic: JWT, signin, refresh, signout |
| `auth.controller.ts` | HTTP handlers with Zod validation |
| `auth.route.ts` | Express router + DI wiring |
| `auth.middleware.ts` | `requireAuth` middleware factory |

## Troubleshooting

### "Invalid token" on refresh

1. Check `JWT_REFRESH_SECRET` matches between token generation and verification
2. Verify refresh token wasn't modified on client side
3. Ensure bcrypt comparison uses the unhashed token from client

### Token expires immediately

1. Check `JWT_ACCESS_EXPIRE` — must be in seconds, not milliseconds
2. Verify system clock is synchronized (NTP)

### "User not found" on signin

1. Check `findUserForAuthentication` includes the `loginId` field correctly
2. Verify query uses correct field (`loginId` vs `email` vs `username`)

## Related Files

For full implementation examples, see:
- `references/prisma-schema.md` — Complete Prisma schema with relations
- `references/shared-types.md` — TypeScript types and Zod schemas