# HTTP Layer 코드 예제

## {module}.controller.ts

```typescript
// modules/user/infrastructure/http/user.controller.ts

import { Request, Response, NextFunction } from 'express';
import { UserService } from '../../application/user.service';
import {
  createUserSchema,
  updateUserSchema,
  userIdParamSchema,
} from '../../domain/user.validation';

export class UserController {
  constructor(private readonly userService: UserService) {}

  // ✅ arrow function 사용 — `this` 바인딩 안전 보장
  getAll = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const users = await this.userService.getAllUsers();
      res.status(200).json({ success: true, data: users });
    } catch (error) {
      next(error);
    }
  };

  getById = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const { id } = userIdParamSchema.parse(req.params);
      const user = await this.userService.getUserById(id);
      res.status(200).json({ success: true, data: user });
    } catch (error) {
      next(error);
    }
  };

  create = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const dto = createUserSchema.parse(req.body);
      const user = await this.userService.createUser(dto);
      res.status(201).json({ success: true, data: user });
    } catch (error) {
      next(error);
    }
  };

  update = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const { id } = userIdParamSchema.parse(req.params);
      const dto = updateUserSchema.parse(req.body);
      const user = await this.userService.updateUser(id, dto);
      res.status(200).json({ success: true, data: user });
    } catch (error) {
      next(error);
    }
  };

  delete = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const { id } = userIdParamSchema.parse(req.params);
      await this.userService.deleteUser(id);
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  };
}
```

### 작성 규칙

- 모든 핸들러는 **arrow function** (`getAll = async (...) => {}`)으로 정의
  - 클래스 메서드를 라우터에 직접 전달할 때 `this` 컨텍스트 유실 방지
- 요청 파싱은 Zod 스키마 사용 (`schema.parse(req.body)` / `schema.parse(req.params)`)
- 비즈니스 로직 없음 — 서비스에 위임
- 에러는 반드시 `next(error)`로 전달

---

## Prisma 싱글톤 (`lib/prisma.ts`)

`PrismaClient`는 앱 전체에서 단일 인스턴스를 공유합니다. Route 파일에서 직접 생성하지 않습니다.

```typescript
// lib/prisma.ts

import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export default prisma;
```

---

## {module}.route.ts

```typescript
// modules/user/infrastructure/http/user.route.ts

import { Router } from 'express';
import prisma from '../../../../lib/prisma';
import { UserRepository } from '../persistence/user.repository';
import { UserService } from '../../application/user.service';
import { UserController } from './user.controller';

// ─── Dependency Injection 조립 ────────────────────────────────────────────────
const userRepository = new UserRepository(prisma);
const userService = new UserService(userRepository);
const userController = new UserController(userService);

// ─── Router ───────────────────────────────────────────────────────────────────
const userRouter = Router();

userRouter.get('/', userController.getAll);
userRouter.get('/:id', userController.getById);
userRouter.post('/', userController.create);
userRouter.patch('/:id', userController.update);
userRouter.delete('/:id', userController.delete);

export default userRouter;
```

### 작성 규칙

- `PrismaClient`는 `lib/prisma.ts` 싱글톤에서 import — Route 파일에서 `new PrismaClient()` 직접 생성 금지
- Route 파일에서 Repository → Service → Controller 순으로 DI 조립
- 미들웨어(인증, 권한 등)가 필요하면 라우터 등록 시 체이닝
  ```typescript
  userRouter.patch('/:id', authenticate, authorize('admin'), userController.update);
  ```
- 컨트롤러 핸들러를 라우터에 등록할 때 `userController.getAll` (괄호 없이) 전달
  - arrow function이므로 별도 `.bind()` 불필요
