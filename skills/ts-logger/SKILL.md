---
name: ts-logger
description: >
  Generates a shared TypeScript logger utility for any project in the init-ts-* family.
  In monorepo mode, writes to packages/core/src/lib/logger.ts and re-exports via the core
  package entry point so all sub-packages can import it. In standalone mode, writes to
  src/lib/logger.ts. Supports debug / log / warn / error levels; debug is suppressed in
  production. Use this skill whenever the user asks to "add a logger", "create a logger
  utility", "replace console.log", "add shared logging", "ts-logger", or whenever an
  init-ts-* command finishes scaffolding and needs logger wiring.
---

# ts-logger

Generates a type-safe, production-aware logger utility for TypeScript projects.

---

## PHASE 1: Detect context

Walk upward from CWD until the filesystem root, looking for `pnpm-workspace.yaml`.

```
Found pnpm-workspace.yaml → MONOREPO mode
Not found                 → STANDALONE mode
```

---

## PHASE 2: Resolve output path

| Mode | Output path | Import path for consumers |
|------|-------------|---------------------------|
| MONOREPO | `packages/core/src/lib/logger.ts` | `@{scope}/core` (re-exported via core `src/index.ts`) |
| STANDALONE | `src/lib/logger.ts` | `@/lib/logger` |

**MONOREPO only** — check whether `packages/core/src/index.ts` already exports the logger.
If not, append the following line (preserve all existing content):

```typescript
export * from './lib/logger.js';
```

---

## PHASE 3: Write logger.ts

Use `write_file` only — no shell redirects (`echo`, heredoc).

```typescript
/**
 * Structured logger utility.
 *
 * Levels   : debug | log | warn | error
 * Rules    : debug output is suppressed when NODE_ENV === 'production'
 *            All output is routed through the matching console method.
 *            Use this module instead of bare console calls in application code.
 */

export type LogLevel = 'debug' | 'log' | 'warn' | 'error';

export interface LogEntry {
  level: LogLevel;
  message: string;
  context?: string;
  data?: unknown;
  timestamp: string;
}

export interface Logger {
  debug(message: string, data?: unknown): void;
  log(message: string, data?: unknown): void;
  warn(message: string, data?: unknown): void;
  error(message: string, data?: unknown): void;
}

// --------------------------------------------------------------------------
// Internal helpers
// --------------------------------------------------------------------------

function isProd(): boolean {
  return (
    typeof process !== 'undefined' && process.env['NODE_ENV'] === 'production'
  );
}

function buildEntry(
  level: LogLevel,
  message: string,
  context: string | undefined,
  data: unknown,
): LogEntry {
  return {
    level,
    message,
    ...(context !== undefined && { context }),
    ...(data !== undefined && { data }),
    timestamp: new Date().toISOString(),
  };
}

function emit(entry: LogEntry): void {
  const prefix = entry.context != null ? `[${entry.context}] ` : '';
  const base = `${entry.timestamp} ${entry.level.toUpperCase()} ${prefix}${entry.message}`;

  if (entry.data !== undefined) {
    // eslint-disable-next-line no-console
    console[entry.level](base, entry.data);
  } else {
    // eslint-disable-next-line no-console
    console[entry.level](base);
  }
}

// --------------------------------------------------------------------------
// Public API
// --------------------------------------------------------------------------

/**
 * Create a named logger bound to a context label (e.g. module or class name).
 *
 * @example
 * const logger = createLogger('UserService');
 * logger.log('User created', { id: user.id });
 */
export function createLogger(context?: string): Logger {
  return {
    debug(message: string, data?: unknown): void {
      if (isProd()) return;
      emit(buildEntry('debug', message, context, data));
    },

    log(message: string, data?: unknown): void {
      emit(buildEntry('log', message, context, data));
    },

    warn(message: string, data?: unknown): void {
      emit(buildEntry('warn', message, context, data));
    },

    error(message: string, data?: unknown): void {
      emit(buildEntry('error', message, context, data));
    },
  };
}

/**
 * Default logger with no context label.
 * Use for top-level scripts or cases where a named logger is unnecessary.
 */
export const logger: Logger = createLogger();
```

---

## PHASE 4: Print usage summary

After saving the file, output the following:

```
logger created

  File   : {RESOLVED_PATH}
  [MONOREPO] core re-export : packages/core/src/index.ts ✅

Usage
─────────────────────────────────────────

  // Default logger (no context label)
  import { logger } from '{IMPORT_PATH}';
  logger.log('Server started');
  logger.warn('Rate limit approaching', { remaining: 3 });
  logger.error('DB connection failed', error);

  // Named logger (recommended — attaches a context prefix)
  import { createLogger } from '{IMPORT_PATH}';
  const logger = createLogger('UserService');
  logger.debug('Running query', { sql });   // suppressed in production
  logger.log('User created', { id });

Level rules
─────────────────────────────────────────
  debug  — dev only; suppressed when NODE_ENV=production
  log    — general info (server start, request handling, etc.)
  warn   — abnormal but recoverable situation
  error  — exceptions, failures, external service errors

  Never use bare console.log / console.error in application code — use logger instead.
```

---

## PHASE 5: ESLint no-console check (optional)

If the project uses ESLint and has a `no-console` rule enabled, verify that `logger.ts`
itself is exempted. If the following override is absent from `eslint.config.js`, recommend
adding it:

```javascript
{
  files: ['**/lib/logger.ts'],
  rules: { 'no-console': 'off' },
},
```

When using `typescript-eslint` with `strictTypeChecked`, `no-console` is off by default,
so the inline `// eslint-disable-next-line no-console` comments in the generated file
are sufficient.

---

## Constraints

- Use `write_file` only — `echo` and heredoc are forbidden.
- If the file already exists, confirm with the user before overwriting.
- In MONOREPO mode, only append to `packages/core/src/index.ts` — never truncate existing content.
- `console` calls are allowed only inside `logger.ts` itself and must include the
  `// eslint-disable-next-line no-console` comment.
