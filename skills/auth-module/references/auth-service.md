# Complete Auth Service Implementation

Full implementation of `auth.service.ts` with token generation, signin, refresh, and signout flows.

```typescript
import { SignJWT, jwtVerify } from "jose";
import bcrypt from "bcrypt";
import type { IAuthRepository } from "./domain/auth.repository.interface";
import type { JWTPayload } from "@shared-types/common/jwt-payload";
import type { AuthUser } from "@shared-types/auth";

export class AuthService {
  constructor(private authRepository: IAuthRepository) {}

  /**
   * Generate access/refresh token pair
   * @private - Internal use only
   */
  private async _generateToken(payload: JWTPayload): Promise<{
    accessToken: string;
    refreshToken: string;
  }> {
    const curTime = new Date().getTime() / 1000;
    const textEncoder = new TextEncoder();

    const accessToken = await new SignJWT({ ...payload })
      .setProtectedHeader({ alg: "HS256" })
      .setExpirationTime(
        parseInt(process.env.JWT_ACCESS_EXPIRE || "60") + curTime
      )
      .sign(textEncoder.encode(process.env.JWT_ACCESS_SECRET));

    const refreshToken = await new SignJWT({ ...payload })
      .setProtectedHeader({ alg: "HS256" })
      .setExpirationTime(
        parseInt(process.env.JWT_REFRESH_EXPIRE || "1209600") + curTime
      )
      .sign(textEncoder.encode(process.env.JWT_REFRESH_SECRET));

    return { accessToken, refreshToken };
  }

  /**
   * Verify access token and return payload
   * @throws null if token is invalid or expired
   */
  public async verifyToken(accessToken: string): Promise<JWTPayload | null> {
    const textEncoder = new TextEncoder();
    try {
      const { payload } = await jwtVerify<JWTPayload>(
        accessToken,
        textEncoder.encode(process.env.JWT_ACCESS_SECRET)
      );
      return payload;
    } catch {
      return null;
    }
  }

  /**
   * Signin flow:
   * 1. Find user by loginId
   * 2. Verify password
   * 3. Check active status
   * 4. Generate tokens
   * 5. Store hashed refresh token
   */
  public async signin(
    loginId: string,
    password: string,
    clientId: string
  ): Promise<{
    accessToken: string;
    refreshToken: string;
    user: AuthUser;
  }> {
    const user = await this.authRepository.findUserForAuthentication(
      loginId,
      clientId
    );

    if (!user) {
      throw new Error("사용자를 찾을 수 없습니다.");
    }

    const isPasswordValid = bcrypt.compareSync(password, user.password);
    if (!isPasswordValid) {
      throw new Error("비밀번호가 일치하지 않습니다.");
    }

    if (user.isUse === false) {
      throw new Error("비활성화된 사용자입니다.");
    }

    const payload: JWTPayload = {
      id: user.id,
      loginId: user.loginId,
      name: user.name,
    };

    const { accessToken, refreshToken } = await this._generateToken(payload);

    // Hash refresh token before storing
    const hashedRefreshToken = bcrypt.hashSync(
      refreshToken,
      parseInt(process.env.JWT_HASH_SALT || "10")
    );

    await this.authRepository.upsertRefreshToken(
      user.id,
      clientId,
      hashedRefreshToken
    );

    return {
      accessToken,
      refreshToken,
      user: {
        id: user.id,
        loginId: user.loginId,
        name: user.name,
      },
    };
  }

  /**
   * Refresh flow:
   * 1. Verify refresh token JWT
   * 2. Find user with stored tokens
   * 3. Compare token against stored hash
   * 4. Generate new token pair
   * 5. Update stored refresh token
   */
  public async refresh(
    refreshToken: string,
    clientId: string
  ): Promise<{
    accessToken: string;
    refreshToken: string;
  }> {
    const textEncoder = new TextEncoder();

    // Step 1: Verify JWT signature and expiration
    let payload: JWTPayload;
    try {
      const result = await jwtVerify<JWTPayload>(
        refreshToken,
        textEncoder.encode(process.env.JWT_REFRESH_SECRET)
      );
      payload = result.payload;
    } catch {
      throw new Error("유효하지 않은 리프레시 토큰입니다.");
    }

    // Step 2: Find user with stored tokens
    const user = await this.authRepository.findUserForAuthentication(
      payload.loginId,
      clientId
    );

    if (!user) {
      throw new Error("사용자를 찾을 수 없습니다.");
    }

    // Step 3: Find matching stored token
    const storedToken = user.tokens.find(
      (t) => t.type === "REFRESH" && t.clientId === clientId
    );

    if (!storedToken) {
      throw new Error("리프레시 토큰을 찾을 수 없습니다.");
    }

    // Step 4: Compare tokens
    const isValid = bcrypt.compareSync(refreshToken, storedToken.token);
    if (!isValid) {
      throw new Error("유효하지 않은 리프레시 토큰입니다.");
    }

    // Step 5: Generate new tokens
    const tokens = await this._generateToken({
      id: user.id,
      loginId: user.loginId,
      name: user.name,
    });

    // Step 6: Update stored token
    const hashedRefreshToken = bcrypt.hashSync(
      tokens.refreshToken,
      parseInt(process.env.JWT_HASH_SALT || "10")
    );

    await this.authRepository.upsertRefreshToken(
      user.id,
      clientId,
      hashedRefreshToken
    );

    return tokens;
  }

  /**
   * Signout: Delete refresh token for this device
   */
  public async signout(userId: string, clientId: string): Promise<void> {
    await this.authRepository.deleteRefreshTokens(userId, clientId);
  }

  /**
   * Signout all devices: Delete all refresh tokens for user
   */
  public async signoutAll(userId: string): Promise<void> {
    // Requires additional repository method:
    // await this.authRepository.deleteAllRefreshTokens(userId);
    throw new Error("Not implemented");
  }
}
```

## Key Implementation Notes

### Token Expiration Calculation

```typescript
// WRONG: Expiration is relative seconds
.setExpirationTime(60) // Invalid - interprets as timestamp

// CORRECT: Add current time for absolute expiration
const curTime = new Date().getTime() / 1000;
.setExpirationTime(parseInt(process.env.JWT_ACCESS_EXPIRE || "60") + curTime)
```

### Why Hash Refresh Tokens?

```typescript
// Store HASHED token in database
const hashedRefreshToken = bcrypt.hashSync(refreshToken, saltRounds);
await repository.upsertRefreshToken(userId, clientId, hashedRefreshToken);

// Verify by COMPARING, not decoding
const isValid = bcrypt.compareSync(providedToken, storedHashedToken);
```

**Security benefit**: If database is compromised, attackers have only hashes — useless without original tokens.

### Why Separate Access/Refresh Secrets?

```typescript
// DIFFERENT secrets for different token types
const accessToken = await new SignJWT(payload)
  .sign(textEncoder.encode(process.env.JWT_ACCESS_SECRET));

const refreshToken = await new SignJWT(payload)
  .sign(textEncoder.encode(process.env.JWT_REFRESH_SECRET));
```

**Security benefit**: Compromised access token cannot forge refresh tokens. Compromised refresh token secret doesn't allow access token forgery.

### Upsert Pattern

```typescript
await this.authRepository.upsertRefreshToken(userId, clientId, hashedToken);
```

This enables **token rotation** — each refresh replaces the old token for that clientId, automatically invalidating previous refresh tokens.

## Error Handling Patterns

```typescript
// Throw errors with user-friendly messages
throw new Error("사용자를 찾을 수 없습니다.");  // Localized for your user base

// Controller catches and transforms
catch (error) {
  if (error instanceof Error) {
    res.status(401).json({ ok: false, message: error.message });
  }
}
```

## Testing the Service

```typescript
import { describe, it, expect, vi } from "vitest";
import { AuthService } from "./auth.service";

describe("AuthService", () => {
  const mockRepo = {
    findUserForAuthentication: vi.fn(),
    upsertRefreshToken: vi.fn(),
    deleteRefreshTokens: vi.fn(),
  };

  const authService = new AuthService(mockRepo);

  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe("signin", () => {
    it("should throw if user not found", async () => {
      mockRepo.findUserForAuthentication.mockResolvedValue(null);
      
      await expect(
        authService.signin("unknown", "pass", "client")
      ).rejects.toThrow("사용자를 찾을 수 없습니다.");
    });

    it("should throw if password invalid", async () => {
      mockRepo.findUserForAuthentication.mockResolvedValue({
        id: "1",
        loginId: "user",
        password: bcrypt.hashSync("correct", 10),
        name: "User",
        isUse: true,
        tokens: [],
      });

      await expect(
        authService.signin("user", "wrong", "client")
      ).rejects.toThrow("비밀번호가 일치하지 않습니다.");
    });

    it("should return tokens on success", async () => {
      const hashedPassword = bcrypt.hashSync("password", 10);
      mockRepo.findUserForAuthentication.mockResolvedValue({
        id: "1",
        loginId: "user",
        password: hashedPassword,
        name: "User",
        isUse: true,
        tokens: [],
      });
      mockRepo.upsertRefreshToken.mockResolvedValue(undefined);

      const result = await authService.signin("user", "password", "client");

      expect(result.accessToken).toBeDefined();
      expect(result.refreshToken).toBeDefined();
      expect(result.user.id).toBe("1");
    });
  });
});
```