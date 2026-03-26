# Persistence Layer 코드 예제

## {module}.repository.ts

```typescript
// modules/user/infrastructure/persistence/user.repository.ts

import { PrismaClient } from '@prisma/client';
import {
  IUserRepository,
  CreateUserDto,
  UpdateUserDto,
  UserEntity,
} from '../../domain/user.repository.interface';
import { User } from '@prisma/client';
import * as bcrypt from 'bcrypt';

export class UserRepository implements IUserRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string): Promise<UserEntity | null> {
    const user = await this.prisma.user.findUnique({
      where: { id },
      select: {
        id: true,
        email: true,
        name: true,
        createdAt: true,
        updatedAt: true,
        // password 필드는 제외
      },
    });
    return user;
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } });
  }

  async findAll(): Promise<UserEntity[]> {
    return this.prisma.user.findMany({
      select: {
        id: true,
        email: true,
        name: true,
        createdAt: true,
        updatedAt: true,
      },
    });
  }

  async create(dto: CreateUserDto): Promise<UserEntity> {
    const hashedPassword = await bcrypt.hash(dto.password, 10);
    const { password, ...userEntity } = await this.prisma.user.create({
      data: {
        email: dto.email,
        name: dto.name,
        password: hashedPassword,
      },
    });
    return userEntity;
  }

  async update(id: string, dto: UpdateUserDto): Promise<UserEntity> {
    const data: Partial<UpdateUserDto> = { ...dto };
    if (dto.password) {
      data.password = await bcrypt.hash(dto.password, 10);
    }
    const { password, ...userEntity } = await this.prisma.user.update({
      where: { id },
      data,
    });
    return userEntity;
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } });
  }
}
```

### 작성 규칙

- `implements I{Module}Repository` 반드시 명시
- 생성자에서 `PrismaClient` 주입 (`private readonly prisma: PrismaClient`)
- 비즈니스 로직 없음 — 오직 DB 접근만
- 민감 정보(password 등)는 select 또는 구조 분해로 제외
