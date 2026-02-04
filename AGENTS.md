# AI Agent Guidelines for Outline

This document provides comprehensive guidance for AI agents working on the Outline codebase. It covers best practices, testing requirements, code review checklists, common pitfalls, file organization, and security considerations.

## Table of Contents

- [Overview](#overview)
- [Quick Reference](#quick-reference)
- [AI Agent Best Practices](#ai-agent-best-practices)
- [Testing Requirements](#testing-requirements)
- [Code Review Checklist](#code-review-checklist)
- [Common Pitfalls](#common-pitfalls)
- [File Organization](#file-organization)
- [Security Considerations](#security-considerations)
- [Development Commands](#development-commands)

---

## Overview

Outline is a fast, collaborative knowledge base built for teams. It is a TypeScript monorepo containing:

- **Frontend**: React web application with MobX state management (`app/`)
- **Backend**: Koa API server with Sequelize ORM, PostgreSQL, and Redis (`server/`)
- **Real-time**: WebSocket-based collaboration using Y.js
- **Editor**: Prosemirror-based rich text editor (`shared/editor/`)
- **Plugins**: Extensible plugin system (`plugins/`)

### Key Dependencies

| Package                 | Purpose                 |
| ----------------------- | ----------------------- |
| `react`                 | UI framework            |
| `mobx` / `mobx-react`   | State management        |
| `koa`                   | Backend web framework   |
| `sequelize`             | PostgreSQL ORM          |
| `ioredis`               | Redis client            |
| `prosemirror-*`         | Rich text editor        |
| `yjs` / `y-prosemirror` | Real-time collaboration |
| `styled-components`     | CSS-in-JS styling       |
| `zod`                   | Schema validation       |
| `bull`                  | Background job queue    |

---

## Quick Reference

### Import Aliases

| Alias       | Maps To      | Usage                    |
| ----------- | ------------ | ------------------------ |
| `@server/*` | `./server/*` | Backend code             |
| `@shared/*` | `./shared/*` | Shared utilities/types   |
| `~/*`       | `./app/*`    | Frontend code            |

### Essential Commands

```bash
yarn lint              # Run linting (oxlint)
yarn tsc               # Type check
yarn test              # Run all tests
yarn test:server       # Server tests only
yarn test:app          # Frontend tests only
yarn format            # Format code with Prettier
```

---

## AI Agent Best Practices

### Before Making Changes

1. **Understand the architecture**: Read `docs/ARCHITECTURE.md` for system design
2. **Check existing patterns**: Look at similar files before creating new ones
3. **Verify import aliases**: Always use `@server/`, `@shared/`, or `~/` instead of relative paths
4. **Review the schema**: Check `shared/validations.ts` for shared validation patterns

### When Writing Code

1. **Follow TypeScript patterns**:
   - No `any` type - use proper types or generics
   - Prefer `interface` over `type` for object shapes
   - Use `import type` for type-only imports
   - Handle null/undefined cases explicitly

2. **Match existing code style**:
   - Functional components with hooks (no class components)
   - Event handlers prefixed with "handle" (e.g., `handleClick`)
   - JSDoc comments on all exported functions
   - Use styled-components for React styling

3. **Follow the command pattern** for business logic:
   - Complex operations go in `server/commands/`
   - Commands should be pure functions with clear inputs/outputs
   - Example: `documentCreator`, `documentUpdater`, `documentMover`

### After Making Changes

1. **Run linting**: `yarn lint` (must pass before commit)
2. **Run type check**: `yarn tsc` (must pass before commit)
3. **Run relevant tests**: `yarn test path/to/file.test.ts`
4. **Verify formatting**: `yarn format:check`

### Commit Guidelines

Use conventional commit format:
- `feat:` New feature
- `fix:` Bug fix
- `refactor:` Code restructuring
- `docs:` Documentation changes
- `test:` Test additions/modifications
- `chore:` Maintenance tasks

---

## Testing Requirements

### Test Configuration

Tests use Jest with the following coverage thresholds:

| Metric     | Threshold |
| ---------- | --------- |
| Statements | 50%       |
| Branches   | 40%       |
| Functions  | 50%       |
| Lines      | 50%       |

### Test File Location

**IMPORTANT**: Tests are colocated with source files. Do NOT create separate test directories.

```
server/models/User.ts
server/models/User.test.ts    # ✓ Correct - next to source

server/models/__tests__/      # ✗ Wrong - don't create test directories
```

### Running Tests

```bash
# Run a specific test file (preferred for development)
yarn test path/to/file.test.ts

# Run all tests
yarn test

# Run test suites by environment
yarn test:app      # Frontend tests (jsdom environment)
yarn test:server   # Backend tests (node environment)
yarn test:shared   # Shared code tests (both environments)
```

### Test Environment Requirements

- **Server tests** require PostgreSQL (CI provides this)
- **Frontend tests** run in jsdom
- Set `TZ=UTC` for consistent date handling (already configured in scripts)

### What Needs Testing

| Change Type          | Required Tests                                    |
| -------------------- | ------------------------------------------------- |
| New API endpoint     | Route handler unit tests, integration tests       |
| New React component  | Rendering tests, interaction tests                |
| Business logic       | Unit tests for commands/utilities                 |
| Database model       | Model validation tests, association tests         |
| Bug fix              | Regression test proving the fix                   |
| Utility function     | Unit tests with edge cases                        |

### Writing Tests

```typescript
// Server test example
import { buildUser, buildDocument } from "@server/test/factories";
import { getTestServer } from "@server/test/support";

const server = getTestServer();

describe("documents.info", () => {
  it("should return document", async () => {
    const user = await buildUser();
    const document = await buildDocument({ userId: user.id });

    const res = await server.post("/api/documents.info", {
      body: { id: document.id },
      headers: { authorization: `Bearer ${user.getJwtToken()}` },
    });

    expect(res.status).toBe(200);
    expect(res.body.data.id).toBe(document.id);
  });
});
```

---

## Code Review Checklist

### TypeScript

- [ ] No use of `any` type
- [ ] `unknown` only used when absolutely necessary
- [ ] `interface` preferred over `type` for object shapes
- [ ] Type assertions (`as`, `!`) minimized
- [ ] Strict null checks respected
- [ ] Consistent `import type` for type-only imports

### React Components

- [ ] Functional components with hooks (no class components)
- [ ] Event handlers prefixed with "handle"
- [ ] styled-components used for styling
- [ ] Accessibility considered (ARIA roles, semantic HTML)
- [ ] `React.memo`, `useMemo`, `useCallback` used where appropriate
- [ ] Self-closing tags for empty elements
- [ ] No unnecessary `import React from "react"`

### Code Style

- [ ] Early returns for readability
- [ ] Curly braces always used for if statements
- [ ] Named exports for components and classes
- [ ] JSDoc for all public/exported functions
- [ ] No `console.log` in production code
- [ ] Arrow body style follows "as-needed" convention
- [ ] Strict equality (`===`) instead of loose equality (`==`)
- [ ] Path aliases used (`@server/`, `@shared/`, `~/`)

### API Routes

- [ ] Request data validated with Zod schema
- [ ] Responses formatted through presenters
- [ ] User authorization checked via policies
- [ ] Errors handled gracefully with proper error types
- [ ] Rate limiting applied where appropriate

### Database

- [ ] Migrations are backward compatible
- [ ] Rollback migration provided
- [ ] Indexes added for frequently queried columns
- [ ] Foreign key constraints properly defined
- [ ] Transactions used for multi-model operations

### Tests

- [ ] Tests added for new functionality
- [ ] Tests colocated with source files
- [ ] Coverage thresholds maintained
- [ ] Edge cases covered
- [ ] Mocks placed in `__mocks__/` folders

---

## Common Pitfalls

### 1. Direct MobX State Mutation

```typescript
// ✗ Wrong - mutating observable directly
user.name = "New Name";

// ✓ Correct - use MobX action
@action
updateName(name: string) {
  this.name = name;
}
```

### 2. Missing Error Handling

```typescript
// ✗ Wrong - unhandled promise
async function fetchData() {
  const data = await api.get("/data");
  return data;
}

// ✓ Correct - proper error handling
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

### 3. Importing React Unnecessarily

```typescript
// ✗ Wrong - JSX transform handles this
import React from "react";

function Component() {
  return <div>Hello</div>;
}

// ✓ Correct - no React import needed
function Component() {
  return <div>Hello</div>;
}
```

### 4. Using Relative Imports

```typescript
// ✗ Wrong - deep relative imports
import { helper } from "../../../shared/utils/helper";

// ✓ Correct - use path aliases
import { helper } from "@shared/utils/helper";
```

### 5. Missing JSDoc on Public Functions

```typescript
// ✗ Wrong - no documentation
export function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// ✓ Correct - proper JSDoc
/**
 * Calculates the total price of all items.
 *
 * @param items - The items to sum.
 * @returns The total price.
 */
export function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}
```

### 6. Using Loose Equality

```typescript
// ✗ Wrong - loose equality
if (value == null) { ... }

// ✓ Correct - strict equality
if (value === null || value === undefined) { ... }
```

### 7. Missing Curly Braces

```typescript
// ✗ Wrong - no curly braces
if (condition) return value;

// ✓ Correct - always use curly braces
if (condition) {
  return value;
}
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

### 9. Forgetting to Use Presenters

```typescript
// ✗ Wrong - returning model directly
ctx.body = { data: document };

// ✓ Correct - use presenter
ctx.body = { data: await presentDocument(ctx, document) };
```

### 10. Skipping Authorization Checks

```typescript
// ✗ Wrong - no authorization
router.post("documents.update", auth(), async (ctx) => {
  const document = await Document.findByPk(id);
  // directly updating...
});

// ✓ Correct - check policy first
router.post("documents.update", auth(), async (ctx) => {
  const document = await Document.findByPk(id);
  authorize(ctx.state.auth.user, "update", document);
  // now safe to update...
});
```

---

## File Organization

### Frontend (`app/`)

| Directory     | Purpose                                           | When to Use                              |
| ------------- | ------------------------------------------------- | ---------------------------------------- |
| `actions/`    | Reusable actions (navigate, open, create)         | Cross-component operations               |
| `components/` | Reusable UI components                            | Shared UI elements                       |
| `editor/`     | Editor-specific React components                  | Editor functionality                     |
| `hooks/`      | Custom React hooks                                | Reusable stateful logic                  |
| `menus/`      | Context menus                                     | Right-click/dropdown menus               |
| `models/`     | MobX observable state models                      | Client-side data models                  |
| `routes/`     | Route definitions (async loaded)                  | New pages/views                          |
| `scenes/`     | Full-page view components                         | Complete page layouts                    |
| `stores/`     | Collections of models + fetch logic               | Data fetching/caching                    |
| `types/`      | TypeScript types                                  | Frontend-specific types                  |
| `utils/`      | Frontend utilities                                | Helper functions                         |

### Backend (`server/`)

| Directory        | Purpose                              | When to Use                              |
| ---------------- | ------------------------------------ | ---------------------------------------- |
| `routes/api/`    | API route handlers                   | New API endpoints                        |
| `routes/auth/`   | Authentication routes                | OAuth/auth providers                     |
| `commands/`      | Business logic commands              | Complex multi-model operations           |
| `config/`        | Database configuration               | DB settings                              |
| `emails/`        | Transactional email templates        | Email notifications                      |
| `middlewares/`   | Koa middleware                       | Request processing                       |
| `migrations/`    | Database migrations                  | Schema changes                           |
| `models/`        | Sequelize database models            | Data models                              |
| `policies/`      | Authorization logic (cancan)         | Permission rules                         |
| `presenters/`    | API response formatters              | JSON response shaping                    |
| `queues/`        | Async queue definitions              | Background jobs                          |
| `services/`      | Application services                 | Service modules (api, worker)            |
| `test/`          | Test helpers and fixtures            | Test utilities only                      |
| `utils/`         | Backend utilities                    | Helper functions                         |

### Shared (`shared/`)

| Directory      | Purpose                       | When to Use                              |
| -------------- | ----------------------------- | ---------------------------------------- |
| `components/`  | Shared React components       | Used in both frontend and backend        |
| `editor/`      | Prosemirror editor            | Editor core logic                        |
| `i18n/`        | Internationalization          | Translations                             |
| `styles/`      | Global styles and colors      | Theme, colors                            |
| `utils/`       | Shared utility methods        | Cross-environment helpers                |
| `types.ts`     | Common TypeScript types       | Shared type definitions                  |
| `validations.ts` | Validation schemas          | Shared Zod schemas                       |

### Plugins (`plugins/`)

Each plugin follows a standard structure:
```
plugins/
├── slack/
│   ├── client/          # Frontend components
│   ├── server/          # Backend routes/logic
│   └── shared/          # Shared types/utils
├── github/
├── google/
└── ...
```

### API Route Structure

```
server/routes/api/
├── documents/
│   ├── documents.ts     # Route handlers
│   ├── documents.test.ts # Tests
│   └── schema.ts        # Zod validation schemas
├── collections/
├── users/
└── index.ts             # Route registration
```

---

## Security Considerations

### Authentication & Authorization

1. **Always use `auth()` middleware** for authenticated routes
2. **Always use `authorize()` from policies** before performing actions
3. **Never trust client input** - validate everything with Zod schemas
4. **Use policies for complex permission logic** - see `server/policies/`

```typescript
// Example: Proper authorization flow
router.post("documents.update", auth(), async (ctx) => {
  const { id } = ctx.input.body;
  const document = await Document.findByPk(id);

  // ALWAYS authorize before performing action
  authorize(ctx.state.auth.user, "update", document);

  // Now safe to proceed
});
```

### Data Validation

1. **Use Zod schemas** for all API input validation
2. **Schema files go in `schema.ts`** next to route handlers
3. **Use `validate()` middleware** to apply schemas

```typescript
// schema.ts
export const DocumentsUpdateSchema = BaseSchema.extend({
  body: z.object({
    id: z.string().uuid(),
    title: z.string().min(1).max(255),
    text: z.string().optional(),
  }),
});

// documents.ts
router.post(
  "documents.update",
  auth(),
  validate(T.DocumentsUpdateSchema),
  async (ctx) => {
    // ctx.input is now typed and validated
  }
);
```

### Sensitive Data

1. **Never log sensitive data** (passwords, tokens, PII)
2. **Use presenters** to filter sensitive fields from responses
3. **Check for credentials in commits** before pushing
4. **Encrypt sensitive model fields** using sequelize-encrypted

### Rate Limiting

1. **Apply rate limiting** to sensitive endpoints
2. **Use `rateLimiter()` middleware** with appropriate strategy

```typescript
router.post(
  "auth.login",
  rateLimiter(RateLimiterStrategy.TenPerMinute),
  async (ctx) => {
    // Rate-limited endpoint
  }
);
```

### SQL Injection Prevention

1. **Always use Sequelize methods** - never raw SQL with user input
2. **Use parameterized queries** when raw SQL is necessary
3. **Validate and sanitize** all user inputs

### XSS Prevention

1. **Presenters sanitize output** - always use them
2. **React escapes by default** - don't use `dangerouslySetInnerHTML`
3. **Validate URLs** before redirecting

### File Uploads

1. **Validate file types** and sizes
2. **Use `AttachmentHelper`** for file handling
3. **Store files in configured storage** (S3, local, etc.)

### Environment Variables

1. **Never commit secrets** - use `.env` files (gitignored)
2. **Reference `env` module** for environment access
3. **Required variables** are validated at startup

---

## Development Commands

### Daily Development

```bash
# Install dependencies
yarn install

# Start development server (frontend + backend with hot reload)
yarn dev:watch

# Run linting
yarn lint

# Run linting on changed files only
yarn lint:changed

# Format code
yarn format

# Type check
yarn tsc
```

### Testing

```bash
# Run all tests
yarn test

# Run specific test file
yarn test path/to/file.test.ts

# Run test suites
yarn test:app      # Frontend tests (jsdom)
yarn test:server   # Backend tests (node)
yarn test:shared   # Shared code tests
```

### Database

```bash
# Run migrations
yarn db:migrate

# Rollback last migration
yarn db:rollback

# Create new migration
yarn db:create-migration

# Reset database (drop, create, migrate)
yarn db:reset
```

### Building

```bash
# Full production build
yarn build

# Build frontend only
yarn vite:build

# Build backend only
yarn build:server

# Clean build artifacts
yarn clean
```

### CI Checks (run before PR)

```bash
yarn lint --quiet    # Linting must pass
yarn tsc             # Type checking must pass
yarn test            # Tests must pass
yarn format:check    # Formatting must be correct
```

---

## Additional Resources

- **Architecture Details**: `docs/ARCHITECTURE.md`
- **Security Policy**: `docs/SECURITY.md`
- **API Documentation**: https://getoutline.com/developers
- **Contributing Guidelines**: See repository CONTRIBUTING.md
- **Services Overview**: `docs/SERVICES.md`

---

## Pull Request Guidelines

### PR Title Format

Use conventional commits: `type: description`

- `feat: Add new feature`
- `fix: Resolve bug in component`
- `refactor: Improve code structure`
- `docs: Update documentation`
- `test: Add missing tests`
- `chore: Update dependencies`

### PR Description Template

```markdown
## Summary
Brief description of changes

## Motivation
Why this change is needed

## Testing
How the changes were tested

## Screenshots (if applicable)
For UI changes

## Breaking Changes (if any)
Note any breaking changes
```

### Before Submitting

1. [ ] All CI checks pass (`lint`, `tsc`, `test`)
2. [ ] Code follows project patterns
3. [ ] Tests added for new functionality
4. [ ] Documentation updated if needed
5. [ ] No sensitive data in commits
6. [ ] PR title follows conventional commits
