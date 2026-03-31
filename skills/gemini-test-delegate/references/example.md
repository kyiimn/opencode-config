# 위임 블록 → Gemini CLI 명령어 변환 예시

## 입력: OpenCode 위임 블록

```markdown
## 위임: 사용자 조회 서비스 구현

- **작업 대상 파일**:
  - `src/services/userService.ts` — 신규 생성

- **구현 명세**:
  - 역할: userId로 사용자를 조회하고 존재하지 않으면 NotFoundError를 throw
  - 입력: `userId: string`
  - 출력: `Promise<User>`
  - 제약: Repository 레이어만 호출 가능, HTTP 클라이언트 직접 호출 금지

- **참조할 규칙**:
  - RULES.md#service-layer
  - RULES.md#error-handling
```

---

## Step 2에서 읽은 실제 구현 코드

```typescript
// src/services/userService.ts
import { UserRepository } from '../repositories/userRepository'
import { NotFoundError } from '../errors'
import type { User } from '../types'

export class UserService {
  constructor(private readonly userRepo: UserRepository) {}

  async getUser(userId: string): Promise<User> {
    if (!userId) {
      throw new Error('userId is required')
    }

    const user = await this.userRepo.findById(userId)

    if (!user) {
      throw new NotFoundError(`User not found: ${userId}`)
    }

    return user
  }
}
```

---

## Step 3에서 생성된 Gemini CLI 명령어

```bash
gemini --yolo -m gemini-2.5-pro -p "다음 TypeScript 구현에 대한 Vitest 테스트 코드를 작성해주세요.

## 구현 정보

**파일**: src/services/userService.ts
**역할**: userId로 사용자를 조회하고 존재하지 않으면 NotFoundError를 throw
**입력**: userId: string
**출력**: Promise<User>
**제약**: Repository 레이어만 호출 가능, HTTP 클라이언트 직접 호출 금지

## 구현 코드

\`\`\`typescript
import { UserRepository } from '../repositories/userRepository'
import { NotFoundError } from '../errors'
import type { User } from '../types'

export class UserService {
  constructor(private readonly userRepo: UserRepository) {}

  async getUser(userId: string): Promise<User> {
    if (!userId) {
      throw new Error('userId is required')
    }
    const user = await this.userRepo.findById(userId)
    if (!user) {
      throw new NotFoundError(\`User not found: \${userId}\`)
    }
    return user
  }
}
\`\`\`

## 테스트 작성 규칙

### 필수 준수사항
- **Vitest 사용** — Jest 절대 금지
- **테스트 파일 위치**: src/services/userService.test.ts
- **커버리지 목표**: 핵심 비즈니스 로직 80% 이상

### 테스트 구조
\`\`\`typescript
describe('{대상_모듈명}', () => {
  describe('{함수명}', () => {
    it('{대상}_{조건}_{기대결과}', () => {
      // given / when / then
    })
  })
})
\`\`\`

### 커버해야 할 케이스
1. Happy path: 유효한 userId로 User 반환
2. Edge case: userId가 빈 문자열인 경우
3. Error path: 존재하지 않는 userId → NotFoundError

### 모킹 규칙
- UserRepository는 vi.mock() 또는 vi.fn()으로 모킹
- 각 테스트 후 vi.clearAllMocks()

### 금지 패턴
- HTTP 클라이언트 직접 호출 금지
- Repository 외 레이어 직접 의존 금지

## 출력 형식
테스트 파일 전체 코드만 출력. 첫 줄에 파일 경로 주석 명시.
// src/services/userService.test.ts"
```

---

## 예상 Gemini 출력

```typescript
// src/services/userService.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { UserService } from './userService'
import { NotFoundError } from '../errors'
import type { User } from '../types'

const mockUserRepo = {
  findById: vi.fn(),
}

describe('UserService', () => {
  let service: UserService

  beforeEach(() => {
    vi.clearAllMocks()
    service = new UserService(mockUserRepo as any)
  })

  describe('getUser', () => {
    it('getUser_유효한userId_User반환', async () => {
      // given
      const mockUser: User = { id: 'user-1', name: 'Alice' }
      mockUserRepo.findById.mockResolvedValue(mockUser)

      // when
      const result = await service.getUser('user-1')

      // then
      expect(result).toEqual(mockUser)
      expect(mockUserRepo.findById).toHaveBeenCalledWith('user-1')
    })

    it('getUser_빈문자열userId_Error반환', async () => {
      // given & when & then
      await expect(service.getUser('')).rejects.toThrow('userId is required')
    })

    it('getUser_존재하지않는userId_NotFoundError반환', async () => {
      // given
      mockUserRepo.findById.mockResolvedValue(null)

      // when & then
      await expect(service.getUser('nonexistent')).rejects.toThrow(NotFoundError)
    })
  })
})
```

---

## 검증 실행

```bash
npx vitest run src/services/userService.test.ts
```
