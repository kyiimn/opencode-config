---
description: >
  Generate BigInt-safe JSON utilities for TypeScript projects.
  Solves TypeError when Prisma BigInt fields cross API boundaries via JSON.
  Creates the core utility, Express middleware, and fetch client wrapper.
  Updates RULES.md with the #bigint-json section.
---

Generate BigInt-safe JSON serialization infrastructure at `$ARGUMENTS`.

**Problem:** `JSON.stringify(1n)` throws `TypeError: Do not know how to serialize a BigInt`.
`JSON.parse` never produces `bigint`. Prisma `BigInt` columns hit this wall on every API response.

**Approach:** Zero-dependency custom replacer/reviver with a tagged-object wire format.
No external packages — the utility is pure TypeScript that lives in `packages/core` (monorepo)
or `src/lib/` (standalone).

**Wire format:** `{ "__bigint__": "9007199254740993" }` — explicit, lossless, portable.

---

## PHASE 0: Detect context

**Step 1 — Resolve target directory.**
Use `$ARGUMENTS` if provided; otherwise use the current working directory.

**Step 2 — Detect monorepo.**
Traverse parent directories up to filesystem root. Check for `pnpm-workspace.yaml`.

```
Found     → MODE=monorepo
Not found → MODE=standalone
```

**Step 3 — Detect package role.**
Read `$ARGUMENTS/AGENTS.md` (or the nearest `AGENTS.md`). Look for role keywords:

| Keyword in AGENTS.md             | Role assigned  |
| --------------------------------- | -------------- |
| `Express`, `REST API`, `DDD`      | `api`          |
| `React`, `Vite`, `shadcn`         | `frontend`     |
| `Commander`, `CLI`                | `cli`          |
| None of the above                 | `library`      |

Set `ROLE` accordingly. Multiple files are created based on ROLE (see per-phase tables).

**Step 4 — Resolve scope (monorepo only).**
Read root `package.json`. Extract `"name"` → strip leading `@` and trailing path to get scope.
Example: `"name": "my-app"` → scope `my-app` → import `@my-app/core`.

---

## PHASE 1: Create core utility

### File paths

| MODE        | Path                                        |
| ----------- | ------------------------------------------- |
| monorepo    | `packages/core/src/lib/bigint-json.ts`      |
| standalone  | `$ARGUMENTS/src/lib/bigint-json.ts`         |

### Content

```typescript
/**
 * BigInt-safe JSON serialization / deserialization.
 *
 * Wire format: { "__bigint__": "<value_as_string>" }
 *
 * @example
 *   bigintJson.stringify({ id: 9007199254740993n, name: 'Alice' })
 *   // → '{"id":{"__bigint__":"9007199254740993"},"name":"Alice"}'
 *
 *   bigintJson.parse<{ id: bigint; name: string }>('...')
 *   // → { id: 9007199254740993n, name: 'Alice' }
 */

const BIGINT_TAG = '__bigint__' as const;

type BigIntTagged = { readonly [BIGINT_TAG]: string };

function isBigIntTagged(value: unknown): value is BigIntTagged {
  return (
    typeof value === 'object' &&
    value !== null &&
    BIGINT_TAG in value &&
    typeof (value as Record<string, unknown>)[BIGINT_TAG] === 'string'
  );
}

function replacer(_key: string, value: unknown): unknown {
  return typeof value === 'bigint' ? { [BIGINT_TAG]: value.toString() } : value;
}

function reviver(_key: string, value: unknown): unknown {
  return isBigIntTagged(value) ? BigInt(value[BIGINT_TAG]) : value;
}

export const bigintJson = {
  /**
   * Serializes a value to JSON string. bigint values are encoded as tagged objects.
   * Drop-in replacement for JSON.stringify where bigint fields may be present.
   */
  stringify(value: unknown, space?: number): string {
    return JSON.stringify(
      value,
      replacer as Parameters<typeof JSON.stringify>[1],
      space,
    );
  },

  /**
   * Parses a JSON string. Tagged objects are restored as bigint values.
   * Drop-in replacement for JSON.parse where bigint fields may be present.
   */
  parse<T = unknown>(text: string): T {
    return JSON.parse(
      text,
      reviver as Parameters<typeof JSON.parse>[1],
    ) as T;
  },
} as const;
```

---

## PHASE 2: Export from core (monorepo only)

Open `packages/core/src/index.ts`. Add the export if not already present:

```typescript
export * from './lib/bigint-json.js';
```

Do not duplicate if already there.

---

## PHASE 3: Create Express middleware (ROLE=api only)

### File paths

| MODE        | Path                                                                        |
| ----------- | --------------------------------------------------------------------------- |
| monorepo    | `$ARGUMENTS/src/common/infrastructure/http/bigint-json.middleware.ts`       |
| standalone  | `$ARGUMENTS/src/common/infrastructure/http/bigint-json.middleware.ts`       |

### Import path for bigintJson

| MODE        | Import                                    |
| ----------- | ----------------------------------------- |
| monorepo    | `@{scope}/core`                           |
| standalone  | `@/lib/bigint-json`                       |

### Content

```typescript
import type { Request, Response, NextFunction, RequestHandler } from 'express';
import { bigintJson } from '{IMPORT_PATH}';

/**
 * Express middleware that replaces express.json() for routes serving BigInt fields.
 *
 * - Outbound: overrides res.json() to serialize with bigintJson.stringify
 * - Inbound:  parses request body with bigintJson.parse (handles tagged BigInt objects)
 *
 * Usage — replace app.use(express.json()) with:
 *   import { bigintJsonMiddleware } from '@/common/infrastructure/http/bigint-json.middleware';
 *   app.use(bigintJsonMiddleware());
 */
export function bigintJsonMiddleware(): RequestHandler {
  return (req: Request, res: Response, next: NextFunction): void => {
    // Outbound: replace res.json to use bigintJson.stringify
    const originalJson = res.json.bind(res) as (body: unknown) => Response;
    res.json = function (body: unknown): Response {
      if (!res.headersSent) {
        res.setHeader('Content-Type', 'application/json; charset=utf-8');
      }
      return originalJson(JSON.parse(bigintJson.stringify(body)) as unknown);
    };

    // Inbound: parse body with bigintJson reviver when Content-Type is application/json
    const contentType = req.headers['content-type'] ?? '';
    if (contentType.includes('application/json') && req.body === undefined) {
      let raw = '';
      req.on('data', (chunk: Buffer) => {
        raw += chunk.toString('utf-8');
      });
      req.on('end', () => {
        try {
          req.body = bigintJson.parse<Record<string, unknown>>(raw);
        } catch {
          req.body = {};
        }
        next();
      });
      req.on('error', () => {
        req.body = {};
        next();
      });
    } else {
      next();
    }
  };
}
```

> **Integration note:** In `app.ts`, replace `app.use(express.json())` with
> `app.use(bigintJsonMiddleware())`. Do not use both simultaneously.

---

## PHASE 4: Create fetch client wrapper (ROLE=frontend or cli)

### File paths

| MODE        | Path                                  |
| ----------- | ------------------------------------- |
| monorepo    | `$ARGUMENTS/src/lib/bigint-fetch.ts`  |
| standalone  | `$ARGUMENTS/src/lib/bigint-fetch.ts`  |

### Import path for bigintJson

| MODE        | Import                                    |
| ----------- | ----------------------------------------- |
| monorepo    | `@{scope}/core`                           |
| standalone  | `@/lib/bigint-json`                       |

### Content

```typescript
import { bigintJson } from '{IMPORT_PATH}';

/**
 * Thin fetch wrapper that uses bigintJson.parse for response bodies.
 * Use in place of fetch() for API calls that return BigInt fields.
 *
 * @example
 *   // GET
 *   const user = await bigintFetch<User>('/api/users/1');
 *
 *   // POST with BigInt in body
 *   const order = await bigintFetch<Order>('/api/orders', {
 *     method: 'POST',
 *     body: bigintJson.stringify({ productId: 1n, quantity: 2n }),
 *   });
 */
export async function bigintFetch<T>(
  url: string,
  options?: RequestInit,
): Promise<T> {
  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      Accept: 'application/json',
      ...options?.headers,
    },
  });

  if (!response.ok) {
    throw new Error(
      `HTTP ${response.status.toString()}: ${response.statusText} — ${url}`,
    );
  }

  const text = await response.text();
  return bigintJson.parse<T>(text);
}
```

---

## PHASE 5: Write unit tests

### Core utility test

| MODE        | Path                                                    |
| ----------- | ------------------------------------------------------- |
| monorepo    | `packages/core/src/lib/bigint-json.test.ts`             |
| standalone  | `$ARGUMENTS/src/lib/bigint-json.test.ts`                |

```typescript
import { describe, it, expect } from 'vitest';
import { bigintJson } from './bigint-json.js';

describe('bigintJson.stringify', () => {
  it('encodes bigint as tagged object', () => {
    expect(bigintJson.stringify({ id: 1n })).toBe('{"id":{"__bigint__":"1"}}');
  });

  it('preserves values beyond Number.MAX_SAFE_INTEGER', () => {
    const big = 9007199254740993n;
    expect(bigintJson.stringify({ val: big })).toBe(
      `{"val":{"__bigint__":"${big.toString()}"}}`,
    );
  });

  it('passes non-bigint values through unchanged', () => {
    expect(bigintJson.stringify({ a: 1, b: 'hello', c: true, d: null })).toBe(
      '{"a":1,"b":"hello","c":true,"d":null}',
    );
  });

  it('handles nested bigint fields', () => {
    expect(bigintJson.stringify({ user: { id: 42n, score: 100n } })).toBe(
      '{"user":{"id":{"__bigint__":"42"},"score":{"__bigint__":"100"}}}',
    );
  });

  it('handles arrays containing bigint', () => {
    expect(bigintJson.stringify([1n, 2n, 3n])).toBe(
      '[{"__bigint__":"1"},{"__bigint__":"2"},{"__bigint__":"3"}]',
    );
  });
});

describe('bigintJson.parse', () => {
  it('restores tagged object as bigint', () => {
    const result = bigintJson.parse<{ id: bigint }>('{"id":{"__bigint__":"1"}}');
    expect(result.id).toBe(1n);
    expect(typeof result.id).toBe('bigint');
  });

  it('preserves precision for large values', () => {
    const big = 9007199254740993n;
    const result = bigintJson.parse<{ val: bigint }>(bigintJson.stringify({ val: big }));
    expect(result.val).toBe(big);
  });

  it('round-trips a mixed object', () => {
    const original = { id: 42n, name: 'Alice', active: true, score: 99 };
    const result = bigintJson.parse<typeof original>(bigintJson.stringify(original));
    expect(result).toEqual(original);
    expect(typeof result.id).toBe('bigint');
  });

  it('does not affect plain objects that happen to lack the tag', () => {
    const result = bigintJson.parse<{ x: number }>('{"x":123}');
    expect(result.x).toBe(123);
  });
});
```

---

## PHASE 6: Update RULES.md

Locate the project's `RULES.md`:

- Monorepo root: `<root>/RULES.md`
- Standalone: `$ARGUMENTS/RULES.md`

**Step 1** — Add `#bigint-json` to the Section Index block at the top of the file.

```
- [`#bigint-json`](#bigint-json) — BigInt-safe JSON · Express middleware · fetch wrapper
```

**Step 2** — Append the following section at the end of the file.

Use `IMPORT_PATH = @{scope}/core` for monorepo, `@/lib/bigint-json` for standalone.

````markdown
## BIGINT JSON {#bigint-json}

> Apply when any Prisma column or API field uses the `BigInt` type.
> Run the `ts-bigint-json` skill to generate the implementation files.

`JSON.stringify` throws `TypeError` on `bigint` values; `JSON.parse` never produces `bigint`.
All BigInt ↔ JSON serialization must go through `bigintJson` — never use native `JSON.*` directly.

### Wire format

`{ "__bigint__": "9007199254740993" }` — tagged object, lossless, portable to any JSON client.

### Import

```typescript
import { bigintJson } from '{IMPORT_PATH}';
```

### Stringify / Parse

```typescript
const body = bigintJson.stringify({ id: 9007199254740993n, name: 'Alice' });
// → '{"id":{"__bigint__":"9007199254740993"},"name":"Alice"}'

const data = bigintJson.parse<{ id: bigint; name: string }>(body);
// → { id: 9007199254740993n, name: 'Alice' }
```

### API Server — Express

Replace `express.json()` with `bigintJsonMiddleware()` in `app.ts`:

```typescript
import { bigintJsonMiddleware } from '@/common/infrastructure/http/bigint-json.middleware';

// replaces: app.use(express.json())
app.use(bigintJsonMiddleware());
```

### Client — Frontend / CLI

```typescript
import { bigintFetch } from '@/lib/bigint-fetch';
import { bigintJson } from '{IMPORT_PATH}';

// GET — response body deserialized with bigintJson.parse
const user = await bigintFetch<User>('/api/users/1');

// POST — body serialized with bigintJson.stringify
const order = await bigintFetch<Order>('/api/orders', {
  method: 'POST',
  body: bigintJson.stringify({ productId: 1n, quantity: 2n }),
});
```

### Do / Don't

- **Do** use `bigintJson.stringify` / `.parse` for any payload that may contain `bigint`
- **Do** use `bigintJsonMiddleware()` in the Express app instead of `express.json()`
- **Do** use `bigintFetch()` on the client side for endpoints that return BigInt fields
- **Don't** use native `JSON.stringify` / `JSON.parse` for API payloads containing BigInt
- **Don't** cast `bigint` to `Number` — values > 2^53 − 1 lose precision silently
- **Don't** use `.toString()` inline as a workaround — it breaks downstream type contracts
````

---

## PHASE 7: Verify

Run automatically. Do not report completion before running.

```bash
# Monorepo
pnpm --filter @{scope}/core tsc --noEmit
pnpm --filter @{scope}/core vitest run

# Standalone
pnpm tsc --noEmit
pnpm vitest run
```

---

## PHASE 8: Report

```
Context
  Mode         : {monorepo | standalone}
  Role         : {api | frontend | cli | library}
  Scope        : {@{scope} | —}

Files created
  Core utility : {path} ✅
  Middleware   : {path | — (role ≠ api)} ✅
  Fetch wrapper: {path | — (role ≠ frontend/cli)} ✅
  Tests        : {path} ✅

RULES.md updated
  File         : {path} ✅
  Section added: #bigint-json ✅

Verification
  tsc --noEmit : {PASS | FAIL}
  vitest run   : {PASS | FAIL}

Next steps
  API server   : Replace app.use(express.json()) with app.use(bigintJsonMiddleware())
  Client       : Use bigintFetch() for endpoints that return BigInt fields
  Prisma       : Prisma BigInt columns serialize automatically through the middleware
```

---

## Constraints

- Write files with `write_file` only — no `echo` or heredoc.
- In monorepo: utility lives in `packages/core` only. Never place it in individual app packages.
- Export from `packages/core/src/index.ts` — use `export *`, avoid manual re-export lists.
- Never cast `bigint` to `number` anywhere in generated code.
- Never import `bigintJson` via relative paths across package boundaries in a monorepo.
- Do not add `json-bigint`, `superjson`, or any other external JSON library as a dependency.
- Run verification before reporting completion.
