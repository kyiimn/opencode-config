---
name: ts-ip-filter
description: >
  Skill for opencode + oh-my-opencode environments. Generates an IP whitelist
  filter middleware for Express TypeScript projects. Automatically loaded by the
  init-ts-api command when Q-A3 = include. Also use for requests like
  "add IP filter", "allowIP middleware", or "install ip-range-check".
---

# ts-ip-filter

Generates an IP whitelist filter middleware for an Express TypeScript project.

## Context Parameters

This skill expects the following context from the caller:

- `TARGET_DIR` — project root path (e.g. `apps/order-api`)

---

## Step 1: Create File

Create the file below using `write_file`.

### `{TARGET_DIR}/src/common/infrastructure/http/ip-filter.middleware.ts`

```typescript
import { NextFunction, Request, Response } from "express";
import ipRangeCheck from "ip-range-check";

/**
 * IP whitelist middleware.
 * @param allowedCidrs - List of allowed CIDRs (e.g. ["127.0.0.1", "10.0.0.0/8"])
 */
export const allowIP = (allowedCidrs: string[]) => {
  return (req: Request, res: Response, next: NextFunction): void => {
    const clientIp =
      (req.headers["x-forwarded-for"] as string)?.split(",")[0]?.trim() ||
      req.socket.remoteAddress;

    if (!clientIp) {
      res.status(403).json({ ok: false, message: "IP not detected" });
      return;
    }
    if (!ipRangeCheck(clientIp, allowedCidrs)) {
      res
        .status(403)
        .json({ ok: false, message: "IP not allowed", ip: clientIp });
      return;
    }
    next();
  };
};
```

---

## Step 2: package.json Dependency

Ensure the following package is listed under `dependencies` in `package.json`.
Skip if already present.

```jsonc
"ip-range-check": "^0.2.0"
```

---

## Step 3: app.ts Integration

Instruct the user to wire the middleware into `src/app.ts` as follows:

```typescript
// Add import:
import { allowIP } from "@/common/infrastructure/http/ip-filter.middleware";

// Chain on a specific router:
const ADMIN_ALLOWED_IPS = ["::1", "127.0.0.1"];
app.use("/auth", allowIP(ADMIN_ALLOWED_IPS), authRouter);

// Or apply globally before all routes:
// app.use(allowIP(ADMIN_ALLOWED_IPS));
```

---

## Completion Report

```
ip-filter
  file    : {TARGET_DIR}/src/common/infrastructure/http/ip-filter.middleware.ts ✅
  package : ip-range-check (add to package.json dependencies)
  wiring  : import allowIP in src/app.ts and chain onto the target router
```
