# Shared Types for Auth Module

TypeScript interfaces and Zod validation schemas shared between frontend and backend.

## JWT Payload

`packages/shared-types/src/common/jwt-payload.ts`:

```typescript
export interface JWTPayload {
  id: string;
  loginId: string;
  name: string;
  // Add custom fields as needed:
  // roles?: string[];
  // permissions?: string[];
  // tenantId?: string;
}
```

Extend based on your application needs. Keep payload minimal — JWT size affects request overhead.

## Auth User

`packages/shared-types/src/auth/index.ts`:

```typescript
export interface AuthUser {
  id: string;
  loginId: string;
  name: string;
  email?: string | null;
}
```

This is the user object returned from signin/refresh.

## Zod Validation Schemas

`packages/shared-types/src/validation/auth.ts`:

```typescript
import { z } from "zod";

// Signin request validation
export const signinSchema = z.object({
  loginId: z.string().min(1, "아이디를 입력해주세요"),
  password: z.string().min(1, "비밀번호를 입력해주세요"),
  clientId: z.string().min(1, "클라이언트 ID가 필요합니다"),
});

export type SigninInput = z.infer<typeof signinSchema>;

// Refresh token request validation
export const refreshTokenSchema = z.object({
  refreshToken: z.string().min(1, "리프레시 토큰이 필요합니다"),
  clientId: z.string().min(1, "클라이언트 ID가 필요합니다"),
});

export type RefreshTokenInput = z.infer<typeof refreshTokenSchema>;

// Signout request validation
export const signoutSchema = z.object({
  clientId: z.string().min(1, "클라이언트 ID가 필요합니다"),
});

export type SignoutInput = z.infer<typeof signoutSchema>;
```

## Response Types

```typescript
// Signin response
export interface SigninResponse {
  ok: boolean;
  data?: {
    accessToken: string;
    refreshToken: string;
    user: AuthUser;
  };
  message?: string;
}

// Refresh response
export interface RefreshResponse {
  ok: boolean;
  data?: {
    accessToken: string;
    refreshToken: string;
  };
  message?: string;
}

// Error response
export interface AuthErrorResponse {
  ok: false;
  message: string;
}
```

## Usage in Frontend

```typescript
import { signinSchema, type SigninInput } from "@shared-types/validation/auth";

// Form validation
const formData: SigninInput = {
  loginId: username,
  password: password,
  clientId: deviceId,
};

try {
  const validated = signinSchema.parse(formData);
  // Send to API
} catch (error) {
  if (error instanceof z.ZodError) {
    // Handle validation errors
    const messages = error.errors.map(e => e.message);
  }
}
```

## Usage in Backend

```typescript
import { signinSchema } from "@shared-types/validation/auth";

// In controller
signin = async (req: Request, res: Response) => {
  try {
    const { loginId, password, clientId } = signinSchema.parse(req.body);
    // Proceed with authenticated request
  } catch (error) {
    if (error instanceof z.ZodError) {
      res.status(400).json({ 
        ok: false, 
        message: error.errors[0].message 
      });
      return;
    }
    throw error;
  }
};
```

## Generating Client ID

Frontend should generate a unique client ID per device:

```typescript
// Option 1: Persist in localStorage
const getClientId = (): string => {
  const stored = localStorage.getItem("clientId");
  if (stored) return stored;
  
  const newId = crypto.randomUUID();
  localStorage.setItem("clientId", newId);
  return newId;
};

// Option 2: Use device fingerprinting library
import FingerprintJS from "@fingerprintjs/fingerprintjs";

const getClientId = async (): Promise<string> => {
  const fp = await FingerprintJS.load();
  const { visitorId } = await fp.get();
  return visitorId;
};
```