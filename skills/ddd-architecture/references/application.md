# Application Layer 코드 예제

## {module}.service.ts

```typescript
// modules/user/application/user.service.ts

import {
  IUserRepository,
  CreateUserDto,
  UpdateUserDto,
  UserEntity,
} from '../domain/user.repository.interface';

export class UserService {
  // 인터페이스 타입으로 주입 — 구현체(UserRepository)를 직접 참조하지 않음
  constructor(private readonly userRepository: IUserRepository) {}

  async getUserById(id: string): Promise<UserEntity> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new Error(`사용자를 찾을 수 없습니다: ${id}`);
    }
    return user;
  }

  async getAllUsers(): Promise<UserEntity[]> {
    return this.userRepository.findAll();
  }

  async createUser(dto: CreateUserDto): Promise<UserEntity> {
    const existing = await this.userRepository.findByEmail(dto.email);
    if (existing) {
      throw new Error('이미 사용 중인 이메일입니다.');
    }
    return this.userRepository.create(dto);
  }

  async updateUser(id: string, dto: UpdateUserDto): Promise<UserEntity> {
    await this.getUserById(id); // 존재 여부 확인
    return this.userRepository.update(id, dto);
  }

  async deleteUser(id: string): Promise<void> {
    await this.getUserById(id); // 존재 여부 확인
    await this.userRepository.delete(id);
  }
}
```

### 작성 규칙

- 생성자 파라미터 타입은 반드시 **인터페이스** (`IUserRepository`), 구현체 클래스 직접 참조 금지
- `req`, `res`, `next` 등 HTTP 관련 코드 절대 포함 금지
- 비즈니스 규칙 (중복 검사, 존재 여부 확인, 권한 확인 등) 이 레이어에서 처리
- 에러는 명확한 메시지와 함께 `throw new Error(...)` 또는 커스텀 에러 클래스 사용
