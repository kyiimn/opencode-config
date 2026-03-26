# Domain Layer 코드 예제

## {module}.repository.interface.ts

```typescript
// modules/user/domain/user.repository.interface.ts

import { User } from '@prisma/client';

// ─── DTOs ────────────────────────────────────────────────────────────────────

export interface CreateUserDto {
  email: string;
  name: string;
  password: string;
}

export interface UpdateUserDto {
  name?: string;
  password?: string;
}

// ─── Entity ───────────────────────────────────────────────────────────────────
// Prisma 생성 타입을 그대로 사용하거나, 민감 필드 제외한 타입을 정의합니다.

export type UserEntity = Omit<User, 'password'>;

// ─── Repository Interface ────────────────────────────────────────────────────

export interface IUserRepository {
  findById(id: string): Promise<UserEntity | null>;
  findByEmail(email: string): Promise<User | null>;
  findAll(): Promise<UserEntity[]>;
  create(dto: CreateUserDto): Promise<UserEntity>;
  update(id: string, dto: UpdateUserDto): Promise<UserEntity>;
  delete(id: string): Promise<void>;
}
```

---

## {module}.validation.ts

```typescript
// modules/user/domain/user.validation.ts

import { z } from 'zod';

export const createUserSchema = z.object({
  email: z.string().email('유효한 이메일 주소를 입력하세요.'),
  name: z.string().min(2, '이름은 최소 2자 이상이어야 합니다.'),
  password: z.string().min(8, '비밀번호는 최소 8자 이상이어야 합니다.'),
});

export const updateUserSchema = z.object({
  name: z.string().min(2).optional(),
  password: z.string().min(8).optional(),
});

export const userIdParamSchema = z.object({
  id: z.string().uuid('유효한 UUID 형식이어야 합니다.'),
});

// Zod 스키마로부터 DTO 타입 추출 (선택적 활용)
export type CreateUserInput = z.infer<typeof createUserSchema>;
export type UpdateUserInput = z.infer<typeof updateUserSchema>;
```
