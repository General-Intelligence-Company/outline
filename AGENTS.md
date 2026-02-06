# AGENTS.md — AI Agent Guidelines for Outline

> Comprehensive coding standards, review checklists, and operational guidelines for AI agents working on the Outline codebase.

---

## Project Overview

**Outline** is a fast, collaborative knowledge base built for teams. It is an open-source TypeScript monorepo with the following core technology stack:

| Layer            | Technology                                      |
| ---------------- | ----------------------------------------------- |
| Frontend         | React 17, MobX, styled-components, Vite         |
| Backend          | Koa 3, Sequelize 6 (PostgreSQL), ioredis (Redis) |
| Editor           | Prosemirror with Y.js real-time collaboration    |
| Validation       | Zod schemas                                      |
| Background Jobs  | Bull queues                                      |
| Package Manager  | Yarn 4.x                                         |
| Linter           | oxlint (type-aware)                              |
| Test Framework   | Jest                                             |
| Node.js          | >=20.12 <21 or 22                                |

### Repository Structure

```
app/            # React frontend (MobX, styled-components, Vite)
server/         # Koa API server (Sequelize ORM, cancan policies, Bull queues)
shared/         # Code shared between frontend and backend
plugins/        # Plugin system for extending functionality
```

| Directory              | Purpose                                           |
| ---------------------- | ------------------------------------------------- |
| `app/actions/`         | Reusable actions (navigating, opening, creating)  |
| `app/components/`      | Reusable UI components                            |
| `app/editor/`          | Editor-specific React components                  |
| `app/hooks/`           | Custom React hooks                                |
| `app/menus/`           | Context menus                                     |
| `app/models/`          | MobX observable state models                      |
| `app/routes/`          | Route definitions (async loaded with suspense)    |
| `app/scenes/`          | Full-page view components                         |
| `app/stores/`          | Collections of models and fetch logic             |
| `app/types/`           | TypeScript types                                  |
| `app/utils/`           | Frontend utilities                                |
| `server/routes/api/`   | API route handlers                                |
| `server/routes/auth/`  | Authentication routes                             |
| `server/commands/`     | Business logic commands                           |
| `server/config/`       | Database configuration                            |
| `server/emails/`       | Transactional email templates                     |
| `server/middlewares/`  | Koa middleware                                    |
| `server/migrations/`   | Database migrations                               |
| `server/models/`       | Sequelize database models                         |
| `server/onboarding/`   | Onboarding document templates                     |
| `server/policies/`     | Authorization logic (cancan)                      |
| `server/presenters/`   | API response formatters                           |
| `server/queues/`       | Async queue definitions                           |
| `server/services/`     | Application service definitions                   |
| `server/test/`         | Test helpers and fixtures                         |
| `server/utils/`        | Backend utilities                                 |
| `shared/components/`   | Shared React components                           |
| `shared/editor/`       | Prosemirror editor components                     |
| `shared/i18n/`         | Internationalization                              |
| `shared/styles/`       | Global styles and colors                          |
| `shared/utils/`        | Shared utility methods                            |
| `shared/types.ts`      | Common TypeScript types                           |
| `shared/validations.ts`| Validation schemas                                |

---

## AI Agent Guidelines

### Before Every Commit

AI agents **must** run the following checks before committing any changes:

```bash
# 1. Lint the codebase (oxlint, type-aware)
yarn lint

# 2. Type-check the entire project
yarn tsc

# 3. Run unit tests
yarn test

# 4. (If backend changes) Run server tests specifically
yarn test:server
```

All four checks must pass cleanly before a commit is created.

### General Principles

1. **Follow existing patterns** — Before writing new code, examine how similar functionality is implemented elsewhere in the codebase. Match the style, structure, and conventions already in use.
2. **Use path aliases** — Never use deep relative imports. Always use the configured aliases:

   | Alias       | Maps To      |
   | ----------- | ------------ |
   | `@server/*` | `./server/*` |
   | `@shared/*` | `./shared/*` |
   | `~/*`       | `./app/*`    |

3. **Respect module boundaries** — Never import server code in `app/` or vice versa. Shared code belongs in `shared/`.
4. **Use `import type`** for type-only imports.
5. **JSDoc all public/exported functions** with `@param` and `@returns` annotations.
6. **No `console.log`** in production code — use the `Logger` utility instead.
7. **Always use strict equality** (`===` / `!==`), never loose equality (`==` / `!=`).
8. **Always use curly braces** for `if`/`else`/`for`/`while` blocks, even single-line bodies.
9. **Use early returns** for readability and to reduce nesting.

### TypeScript Standards

- **No `any` type** — Use proper types or generics. If `any` is absolutely unavoidable, add a `// eslint-disable-next-line` comment with justification.
- **Avoid `unknown`** — Only use when absolutely necessary.
- **Prefer `interface`** over `type` for object shapes.
- **Avoid type assertions** — Minimize use of `as` and non-null assertion `!` operators.
- **Strict null checks** — Always handle `null`/`undefined` cases explicitly.

```typescript
// ✓ Correct — type-only import with alias
import type { User } from "@server/models/User";

// ✗ Incorrect — runtime import when only used as a type
import { User } from "@server/models/User";
```

### React & Frontend Patterns

- **Functional components only** — Never use class components.
- **No React import** — JSX transform is enabled, `import React` is unnecessary.
- **Event handler naming** — Always prefix with `handle` (e.g., `handleClick`, `handleSubmit`).
- **Styling** — Use `styled-components` for all component styles. Do not use inline styles or CSS modules.
- **Performance** — Use `React.memo`, `useMemo`, `useCallback` to prevent unnecessary re-renders.
- **Self-closing tags** — Always use self-closing tags for empty elements (`<div />` not `<div></div>`).
- **MobX actions** — Always use `@action` decorators for state mutations; use `@computed` for derived state.

### Backend Patterns

- **Validation** — Use validation middleware (Zod schemas in `shared/validations.ts`) for all request data.
- **Presenters** — Always format API responses through presenters (`server/presenters/`).
- **Authorization** — Check user permissions via cancan policies (`server/policies/`) before performing actions.
- **Commands** — Encapsulate complex business logic in commands (`server/commands/`), not inline in route handlers.
- **Error handling** — Handle errors gracefully with proper error types from `http-errors`. Never swallow errors silently.
- **Sequelize models** — Use `sequelize-typescript` decorators (`@Table`, `@Column`, `@BelongsTo`, etc.) for model definitions.

### Editor Patterns

- Editor components live in `shared/editor/`.
- Use Y.js (`y-prosemirror`) for real-time collaboration.
- Follow existing Prosemirror node and mark patterns when adding new editor functionality.

---

## Code Review Checklist

AI agents must verify all of the following before submitting code:

### TypeScript & Code Quality

- [ ] No use of `any` type (unless justified with a comment)
- [ ] `unknown` only used when absolutely necessary
- [ ] `interface` preferred over `type` for object shapes
- [ ] No unnecessary type assertions (`as`, `!`)
- [ ] Strict null checks respected — no unguarded access to possibly-null values
- [ ] Consistent `import type` used for type-only imports
- [ ] JSDoc present on all public/exported functions
- [ ] No `console.log` in production code
- [ ] Strict equality (`===`) used everywhere
- [ ] Curly braces on all control-flow blocks

### Sequelize Models & Database

- [ ] Sequelize models use proper `sequelize-typescript` decorators (`@Table`, `@Column`, `@ForeignKey`, etc.)
- [ ] All database schema changes have corresponding migrations (`server/migrations/`)
- [ ] Migrations are backward compatible
- [ ] Rollback migration logic is provided
- [ ] Indexes added for frequently queried columns
- [ ] Foreign key constraints properly defined

### API Routes & Authentication

- [ ] All API routes have authentication middleware applied
- [ ] Request data validated using validation middleware (Zod schemas)
- [ ] Responses formatted through presenters (`server/presenters/`)
- [ ] User authorization checked via cancan policies (`server/policies/`)
- [ ] Errors handled gracefully with proper HTTP error types

### Frontend Components

- [ ] Functional components with hooks (no class components)
- [ ] `styled-components` used for all styling (no inline styles)
- [ ] Event handlers prefixed with `handle`
- [ ] Accessibility considered (ARIA roles, semantic HTML)
- [ ] Self-closing tags for empty elements
- [ ] No unnecessary React import
- [ ] Performance optimizations applied (`React.memo`, `useMemo`, `useCallback`) where appropriate

### Internationalization (i18n)

- [ ] All user-facing strings wrapped with translation functions (`t()` / `useTranslation`)
- [ ] Translation keys added to `shared/i18n/` locale files
- [ ] No hardcoded English strings in UI components

### Test Coverage

- [ ] Tests added for all new functionality
- [ ] Test files colocated with source files (e.g., `User.test.ts` next to `User.ts`)
- [ ] Coverage thresholds maintained (see Testing Requirements below)
- [ ] No test directories created — tests go next to their source files

---

## Testing Requirements

### Coverage Thresholds

Tests use Jest with the following minimum coverage thresholds:

| Metric     | Threshold |
| ---------- | --------- |
| Statements | 50%       |
| Branches   | 40%       |
| Functions  | 50%       |
| Lines      | 50%       |

### Running Tests

```bash
# Run a specific test file (preferred during development)
yarn test path/to/file.test.ts

# Run all tests
yarn test

# Run by suite
yarn test:app          # Frontend tests (jsdom environment)
yarn test:server       # Backend tests (node environment)
yarn test:shared       # Shared code tests (both environments)
```

### What Needs Testing

| Change Type          | Testing Requirements                                         |
| -------------------- | ------------------------------------------------------------ |
| New API endpoint     | Unit tests for route handler; integration tests using `server/test/` helpers |
| New React component  | Component rendering tests; snapshot tests where applicable   |
| New utility function | Unit tests with edge cases                                   |
| Business logic       | Unit tests for commands/utilities                            |
| Database model       | Model validation tests, association tests                    |
| Bug fix              | Regression test that proves the fix                          |

### Test Patterns

- **Backend tests** — Use test helper patterns from `server/test/` (e.g., factory functions for creating test data, database setup/teardown).
- **Frontend tests** — Run in jsdom environment. Use snapshot tests for UI components where appropriate.
- **Shared tests** — Run in both node and jsdom environments.
- **Mocking** — Mock external dependencies in `__mocks__/` folders. Do not mock internal modules unless necessary.
- **Test file location** — Always colocate tests with source files:
  ```
  server/models/User.ts
  server/models/User.test.ts
  ```
  **Do not create new test directories.**

---

## Common Pitfalls

### 1. Importing Across Module Boundaries

```typescript
// ✗ WRONG — importing server code in the frontend
import { User } from "@server/models/User"; // in app/ code

// ✗ WRONG — importing frontend code in the backend
import { Button } from "~/components/Button"; // in server/ code

// ✓ CORRECT — use shared/ for code needed in both
import { UserRole } from "@shared/types";
```

### 2. Using Relative Imports Instead of Aliases

```typescript
// ✗ Wrong
import { helper } from "../../../shared/utils/helper";

// ✓ Correct
import { helper } from "@shared/utils/helper";
```

### 3. Missing Database Migrations

All database schema changes **must** have a corresponding migration file in `server/migrations/`. Never modify the database schema without creating a migration. Create migrations with:

```bash
yarn db:create-migration --name descriptive-migration-name
```

### 4. Using `any` Without Justification

```typescript
// ✗ Wrong — untyped
function process(data: any) { ... }

// ✓ Correct — properly typed
function process(data: DocumentPayload) { ... }

// ✓ Acceptable — justified escape hatch (rare)
// eslint-disable-next-line @typescript-eslint/no-explicit-any
function legacyAdapter(data: any) { ... } // Required for third-party API compatibility
```

### 5. Direct MobX State Mutation

```typescript
// ✗ Wrong — mutating observable directly
user.name = "New Name";

// ✓ Correct — use a MobX action
@action
updateName(name: string) {
  this.name = name;
}
```

### 6. Missing Error Handling

```typescript
// ✗ Wrong — unhandled async error
async function fetchData() {
  const data = await api.get("/data");
  return data;
}

// ✓ Correct — proper error handling
async function fetchData() {
  try {
    const data = await api.get("/data");
    return data;
  } catch (error) {
    Logger.error("Failed to fetch data", error);
    throw error;
  }
}
```

### 7. Hardcoded User-Facing Strings

```typescript
// ✗ Wrong — hardcoded English string
<p>No documents found</p>

// ✓ Correct — internationalized
const { t } = useTranslation();
<p>{t("No documents found")}</p>
```

### 8. Non-Self-Closing Empty Elements

```tsx
// ✗ Wrong
<div className="spacer"></div>
<Component prop="value"></Component>

// ✓ Correct
<div className="spacer" />
<Component prop="value" />
```

---

## PR Guidelines

### Before Opening a PR

1. Run all checks locally:
   ```bash
   yarn lint          # Must pass
   yarn tsc           # Must pass
   yarn test          # Must pass (frontend + shared)
   yarn test:server   # Must pass (if backend changes)
   ```
2. Ensure all CI jobs will pass: `lint`, `types`, `test`, `test-server`.

### PR Title Format

Use conventional commit format:

- `feat: Add new feature`
- `fix: Resolve bug in component`
- `refactor: Improve code structure`
- `docs: Update documentation`
- `test: Add missing tests`
- `chore: Update dependencies`

### PR Description Requirements

Every PR must include:

1. **Summary** — Brief description of what changed and why.
2. **Related Issues** — Reference related GitHub issues (e.g., `Closes #123`, `Fixes #456`).
3. **Test Plan** — Describe how the changes were tested:
   - Which test commands were run
   - Manual testing steps (if applicable)
   - Screenshots for UI changes
4. **Breaking Changes** — Clearly document any breaking changes, required migrations, or feature flags.

### CI Checks That Must Pass

| CI Job          | Command          | Description                              |
| --------------- | ---------------- | ---------------------------------------- |
| `lint`          | `yarn lint`      | oxlint type-aware linting                |
| `types`         | `yarn tsc`       | TypeScript type checking                 |
| `test`          | `yarn test:app`  | Frontend + shared tests (jsdom)          |
| `test-server`   | `yarn test:server` | Backend tests with PostgreSQL (node)   |

All four checks must pass before a PR can be merged.

---

## Development Environment

### Prerequisites

- Node.js (>=20.12 <21 or 22)
- Yarn 4.x (package manager)
- PostgreSQL
- Redis

### Common Commands

```bash
yarn install              # Install dependencies
yarn dev:watch            # Start dev server (frontend + backend)
yarn lint                 # Run oxlint (type-aware)
yarn format               # Format code with Prettier
yarn tsc                  # TypeScript type check
yarn test                 # Run all tests
yarn test:app             # Frontend tests only
yarn test:server          # Backend tests only
yarn test:shared          # Shared code tests
yarn db:migrate           # Run database migrations
yarn db:rollback          # Rollback last migration
yarn db:create-migration  # Create a new migration file
```

---

## Additional Resources

- See `docs/ARCHITECTURE.md` for detailed system architecture
- API documentation: https://getoutline.com/developers
- Companion file: `CLAUDE.md` contains additional project-specific patterns and style guidance
