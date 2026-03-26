# Prisma Schema for Auth Module

Complete Prisma schema including user and token models.

## Core Models

```prisma
// ============================================
// USER MODEL
// ============================================
model UserInfo {
  id         String      @id @default(cuid())
  loginId    String      @unique @map("login_id") @db.VarChar(50)
  password   String      @db.VarChar(255)  // bcrypt hashed
  name       String      @db.VarChar(100)
  email      String?     @db.VarChar(255)
  isUse      Boolean     @default(true) @map("is_use")
  createdAt  DateTime    @default(now()) @map("created_at")
  updatedAt  DateTime    @updatedAt @map("updated_at")
  
  tokens     UserToken[]

  @@map("user_info")
}

// ============================================
// TOKEN MODEL
// ============================================
model UserToken {
  id        BigInt        @id @default(autoincrement())
  userId    String        @map("user_id")
  clientId  String        @map("client_id") @db.VarChar(255)
  type      UserTokenType @default(REFRESH)
  token     String        @db.VarChar(255)  // Hashed refresh token
  createdAt DateTime      @default(now()) @map("created_at")
  updatedAt DateTime      @updatedAt @map("updated_at")

  user UserInfo @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([userId, type, clientId])  // One token per user/client
  @@map("user_token")
}

// ============================================
// ENUMS
// ============================================
enum UserTokenType {
  ACCESS
  REFRESH
}
```

## Key Design Decisions

### Composite Unique Key

```prisma
@@unique([userId, type, clientId])
```

This enables:
- **Multi-device support**: Different clientIds = different sessions
- **Token rotation**: Upsert replaces old token for same clientId
- **Single token per device**: Only one refresh token per (user, client) pair

### onDelete Cascade

```prisma
user UserInfo @relation(fields: [userId], references: [id], onDelete: Cascade)
```

When a user is deleted, all their tokens are automatically removed.

### Token Types

The `UserTokenType` enum supports future extensibility:
- `REFRESH`: Current use case
- `ACCESS`: Reserved for server-side access token tracking (optional)

## Migration

```bash
# Create migration
pnpm prisma migrate dev --name add_user_token

# Generate client
pnpm prisma generate
```

## Queries Reference

### Find User with Tokens

```typescript
const user = await prisma.userInfo.findUnique({
  where: { loginId: "username" },
  include: {
    tokens: {
      where: { clientId: "device-uuid" }
    }
  }
});
```

### Upsert Token

```typescript
await prisma.userToken.upsert({
  where: {
    userId_type_clientId: {
      userId: "user-id",
      type: "REFRESH",
      clientId: "device-uuid"
    }
  },
  update: { token: "hashed-token" },
  create: {
    userId: "user-id",
    type: "REFRESH",
    clientId: "device-uuid",
    token: "hashed-token"
  }
});
```

### Delete All User Tokens (Full Logout)

```typescript
await prisma.userToken.deleteMany({
  where: { userId: "user-id" }
});
```

### Delete Device Token (Single Device Logout)

```typescript
await prisma.userToken.deleteMany({
  where: { userId: "user-id", clientId: "device-uuid" }
});
```