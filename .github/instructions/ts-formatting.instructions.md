---
description: 'TypeScript formatting standards, type annotations, and ESLint enforcement'
applyTo: '**/*.ts,**/*.tsx'
---

# TypeScript Formatting & Standards

This document defines TypeScript formatting conventions and how they are enforced through ESLint.

## Type Annotations

### Explicit Parameter and Return Types

All exported functions and complex helpers must have **explicit parameter and return types**. This ensures the code is self-documenting and type-safe:

```ts
// Good: explicit types
export async function getAllGameIds(db: Database): Promise<number[]> {
  const rows = await db.select({ id: games.id }).from(games);
  return rows.map((r) => r.id);
}

// Also acceptable for simple functions with obvious types
function sum(a: number, b: number): number {
  return a + b;
}
```

### Avoid `any`

Never use `any`. Use `unknown` if the type is truly unknown, or create a more specific type:

```ts
// Bad
function process(data: any) { }

// Good
function process(data: unknown) {
  if (typeof data === 'string') {
    // ...
  }
}

// Or define a specific type
interface GameData {
  id: number;
  title: string;
}

function process(data: GameData) { }
```

### Unused Variables

Use a leading underscore (`_`) to indicate intentionally unused parameters. ESLint is configured to allow this pattern:

```ts
// Good: underscore indicates we're deliberately not using the parameter
export function middleware(_req: Request, res: Response) {
  res.json({ success: true });
}

// Bad: leftover unused variable suggests incomplete refactoring
export function middleware(req: Request, res: Response) {
  res.json({ success: true });
}
```

## Naming Conventions

### Identifiers

- **Variables & functions:** camelCase (e.g., `getAllGames`, `gameTitle`)
- **Constants:** UPPER_SNAKE_CASE for true constants, camelCase for const variables with no runtime reassignment (e.g., `const config = { ... }`)
- **Types & interfaces:** PascalCase (e.g., `Game`, `Publisher`, `DatabaseConfig`)
- **Files:** kebab-case for components and helpers (e.g., `game-card.astro`, `db-helpers.ts`), PascalCase only for test utilities

### Boolean Variables

Use verb prefixes to make boolean names clear:

```ts
// Good
const isLoading = true;
const hasError = false;
const canEdit = true;

// Avoid (ambiguous)
const loading = true;
const error = false;
```

## ESLint Enforcement

This project uses **ESLint** with `typescript-eslint` to enforce code quality. ESLint runs automatically in CI on pull requests.

### Running ESLint

Use the `quality-checks` skill to run linting (preferred) or run directly:

```bash
npm run lint
```

### Core Rules

The project enforces:

- **`@typescript-eslint/no-unused-vars`** — Variables must be used or prefixed with `_`
- **`@typescript-eslint/explicit-function-return-types`** (recommended for exported functions) — Functions should declare return types
- **`@typescript-eslint/no-explicit-any`** — Avoid `any` in favor of `unknown` or specific types
- ESLint's **`no-var`** — Use `const`/`let`, not `var`

### ESLint Configuration

All ESLint rules are defined in [`eslint.config.js`](../../eslint.config.js) at the repository root. Rules are applied to:

- All TypeScript files (`**/*.ts`)
- All Astro files (`**/*.astro`)
- JavaScript files in the project root and config

To add or modify rules, edit `eslint.config.js` and run `npm run lint` to verify.

## Type Checking

The project runs type checking in two ways:

### TypeScript 7 (tsgo) — Pure TypeScript

```bash
npm run typecheck
```

Checks `db/**/*.ts`, `src/lib/*.ts`, `src/types/*.ts`, and test files against `tsconfig.tsgo.json`. Requires:

- Explicit parameter and return types on exported functions
- No `any` types
- Correct type imports/exports

### Astro Type Checking

```bash
npm run typecheck:astro
```

Checks `.astro` files with the classic TypeScript package via `astro check`. Requires `.astro` files to have properly typed `Props` interfaces and correct data-access imports.

### Run All Type Checks

```bash
npm run typecheck:all
```

Runs both type-check commands. This runs in CI on all pull requests.

## Comment Standards

See [`astro.instructions.md`](astro.instructions.md) and [`drizzle.instructions.md`](drizzle.instructions.md) for comment philosophy:

- Comment *why*, not *what*
- Document intent and non-obvious decisions
- Remove comments that merely paraphrase code
- Keep comments current as you refactor

## Best Practices

- Use `const` by default, `let` when reassignment is needed, never `var`
- Import types explicitly: `import type { Game } from '../types'`
- Use interface for object shapes, type for unions/tuples
- Prefer `interface` for extensible types (props, config)
- Keep types close to usage; define in the file where they're used unless shared across modules
