# AI Agent Guidelines for Outline

This document provides guidance for AI agents working on the Outline codebase.

## Overview

Outline is a fast, collaborative knowledge base built for teams. It is a TypeScript monorepo containing:

- **Frontend**: React web application with MobX state management
- **Backend**: Koa API server with Sequelize ORM, PostgreSQL, and Redis
- **Real-time**: WebSocket-based collaboration using Y.js
- **Editor**: Prosemirror-based rich text editor

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

## Architecture Overview

### Frontend (`app/`)

The frontend is a React application compiled with Vite, using MobX for state management and styled-components for styling.

| Directory          | Purpose                                          |
| ------------------ | ------------------------------------------------ |
| `app/actions/`     | Reusable actions (navigating, opening, creating) |
| `app/components/`  | Reusable UI components                           |
| `app/editor/`      | Editor-specific React components                 |
| `app/hooks/`       | Custom React hooks                               |
| `app/menus/`       | Context menus                                    |
| `app/models/`      | MobX observable state models                     |
| `app/routes/`      | Route definitions (async loaded with suspense)   |
| `app/scenes/`      | Full-page view components                        |
| `app/stores/`      | Collections of models and fetch logic            |
| `app/types/`       | TypeScript types                                 |
| `app/utils/`       | Frontend utilities                               |

### Backend (`server/`)

The API server is built on Koa with Sequelize ORM. Authorization uses cancan policies, and background jobs are managed with Bull queues.

| Directory              | Purpose                               |
| ---------------------- | ------------------------------------- |
| `server/routes/api/`   | API route handlers                    |
| `server/routes/auth/`  | Authentication routes                 |
| `server/commands/`     | Business logic commands               |
| `server/config/`       | Database configuration                |
| `server/emails/`       | Transactional email templates         |
| `server/middlewares/`  | Koa middleware                        |
| `server/migrations/`   | Database migrations                   |
| `server/models/`       | Sequelize database models             |
| `server/onboarding/`   | Onboarding document templates         |
| `server/policies/`     | Authorization logic (cancan)          |
| `server/presenters/`   | API response formatters               |
| `server/queues/`       | Async queue definitions               |
| `server/services/`     | Application service definitions       |
| `server/test/`         | Test helpers and fixtures             |
| `server/utils/`        | Backend utilities                     |

### Shared (`shared/`)

Code shared between frontend and backend.

| Directory             | Purpose                         |
| --------------------- | ------------------------------- |
| `shared/components/`  | Shared React components         |
| `shared/editor/`      | Prosemirror editor components   |
| `shared/i18n/`        | Internationalization            |
| `shared/styles/`      | Global styles and colors        |
| `shared/utils/`       | Shared utility methods          |
| `shared/types.ts`     | Common TypeScript types         |
| `shared/validations.ts` | Validation schemas            |

### Plugins (`plugins/`)

Plugin system for extending functionality.

## Code Style & Patterns

### TypeScript Patterns

- **No `any` type**: Avoid using `any`; use proper types or generics
- **Avoid `unknown`**: Only use when absolutely necessary
- **Prefer `interface`**: Use `interface` over `type` for object shapes
- **Avoid type assertions**: Minimize use of `as` and `!` operators
- **Strict null checks**: Always handle null/undefined cases
- **Consistent type imports**: Use `import type` for type-only imports

```typescript
// ✓ Correct - use consistent type imports
import type { User } from "@server/models/User";

// ✗ Incorrect
import { User } from "@server/models/User"; // when only using as a type
```

### Import Conventions

Use path aliases instead of relative imports:

```typescript
// ✓ Correct
import { User } from "@server/models/User";
import { formatDate } from "@shared/utils/date";
import { Button } from "~/components/Button";

// ✗ Incorrect - avoid deep relative imports
import { User } from "../../../server/models/User";
```

| Alias       | Maps To      |
| ----------- | ------------ |
| `@server/*` | `./server/*` |
| `@shared/*` | `./shared/*` |
| `~/*`       | `./app/*`    |

### MobX State Management Patterns

```typescript
// ✗ Wrong - mutating observable directly
user.name = "New Name";

// ✓ Correct - use MobX action
@action
updateName(name: string) {
  this.name = name;
}
```

- Always use `@action` decorators for state mutations
- Use `@computed` for derived state
- Keep stores focused and single-purpose
- Co-locate state logic with components when not global

### React Component Patterns

- **Functional components**: Always use functional components with hooks
- **Event handler naming**: Prefix with "handle" (e.g., `handleClick`, `handleSubmit`)
- **Styling**: Use styled-components for component styles
- **No React import**: JSX transform is enabled, no need to import React
- **Performance**: Use `React.memo`, `useMemo`, `useCallback` to avoid unnecessary re-renders
- **Self-closing tags**: Always use self-closing tags for empty elements

```typescript
// ✗ Wrong - unnecessary React import
import React from "react";

function Component() {
  return <div>Hello</div>;
}

// ✓ Correct - no React import needed with JSX transform
function Component() {
  return <div>Hello</div>;
}
```

```typescript
// ✗ Wrong - non-self-closing empty element
<div className="spacer"></div>

// ✓ Correct - self-closing
<div className="spacer" />
```

### Koa Backend Patterns

- **Validation**: Use validation middleware for request data
- **Presenters**: Always format API responses through presenters
- **Policies**: Check authorization via cancan policies
- **Error handling**: Handle errors gracefully with proper error types
- **Commands**: Use commands for complex business logic across models

### Prosemirror Editor Patterns

- Editor components are in `shared/editor/`
- Use Y.js for real-time collaboration
- Follow existing node and mark patterns

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

Tests are colocated with source files using `.test.ts` or `.test.tsx` extension:

```
server/models/User.ts
server/models/User.test.ts
```

**Do not create new test directories** - tests belong next to their source files.

### Running Tests

```bash
# Run a specific test file (preferred)
yarn test path/to/file.test.ts

# Run all tests
yarn test

# Run test suites
yarn test:app      # Frontend tests (jsdom environment)
yarn test:server   # Backend tests (node environment)
yarn test:shared   # Shared code tests (both environments)
```

### Writing Tests

- Use Jest for all tests
- Mock external dependencies in `__mocks__/` folders
- Focus on critical paths and business logic
- Frontend tests run in jsdom environment
- Backend tests run in node environment
- Shared tests run in both environments

### What Needs Testing

| Change Type          | Testing Requirements                              |
| -------------------- | ------------------------------------------------- |
| New API endpoint     | Unit tests for route handler, integration tests   |
| New React component  | Component rendering tests, interaction tests      |
| Business logic       | Unit tests for commands/utilities                 |
| Database model       | Model validation tests, association tests         |
| Bug fix              | Regression test proving the fix                   |

## Code Review Checklist

### TypeScript

- [ ] No use of `any` type
- [ ] Avoid `unknown` unless necessary
- [ ] Prefer `interface` over `type` for object shapes
- [ ] Avoid type assertions (`as`, `!`)
- [ ] Strict null checks are respected
- [ ] Consistent type imports used

### React Components

- [ ] Use functional components with hooks
- [ ] Event handlers prefixed with "handle"
- [ ] Use styled-components for styling
- [ ] Ensure accessibility (ARIA roles, semantic HTML)
- [ ] Avoid unnecessary re-renders
- [ ] Self-closing tags for empty elements

### Code Style

- [ ] Use early returns for readability
- [ ] Always use curly braces for if statements
- [ ] Named exports for components and classes
- [ ] JSDoc for all public/exported functions
- [ ] No `console.log` in production code
- [ ] Arrow body style follows "as-needed" convention
- [ ] Strict equality (`===`) used instead of loose equality

### API Routes

- [ ] Validate request data with validation middleware
- [ ] Use presenters for response formatting
- [ ] Check user authorization via policies
- [ ] Handle errors gracefully

### Database

- [ ] Migrations are backward compatible
- [ ] Rollback migration is provided
- [ ] Indexes added for frequently queried columns
- [ ] Foreign key constraints properly defined

### Tests

- [ ] Tests added for new functionality
- [ ] Coverage thresholds maintained
- [ ] Tests colocated with source files

## Common Pitfalls

### 1. Direct State Mutation

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
// ✗ Wrong - unnecessary React import
import React from "react";

function Component() {
  return <div>Hello</div>;
}

// ✓ Correct - no React import needed with JSX transform
function Component() {
  return <div>Hello</div>;
}
```

### 4. Using Relative Imports

```typescript
// ✗ Wrong
import { helper } from "../../../shared/utils/helper";

// ✓ Correct
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
 * @param items - the items to sum.
 * @returns the total price.
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

## Development Environment

### Prerequisites

- Node.js (>=20.12 <21 or 22)
- Yarn 4.x (package manager)
- PostgreSQL
- Redis

### Common Commands

```bash
# Install dependencies
yarn install

# Start development server (frontend + backend)
yarn dev:watch

# Run linting
yarn lint

# Format code
yarn format

# Type check
yarn tsc

# Run all tests
yarn test

# Database migrations
yarn db:migrate           # Run migrations
yarn db:rollback          # Rollback last migration
yarn db:create-migration  # Create new migration
```

## Pull Request Guidelines

### PR Title Format

Use conventional commit format:
- `feat: Add new feature`
- `fix: Resolve bug in component`
- `refactor: Improve code structure`
- `docs: Update documentation`
- `test: Add missing tests`
- `chore: Update dependencies`

### PR Description

Include:
1. **Summary**: Brief description of changes
2. **Motivation**: Why this change is needed
3. **Testing**: How the changes were tested
4. **Screenshots**: For UI changes (if applicable)
5. **Breaking Changes**: Note any breaking changes

### Breaking Changes

- Document breaking changes clearly in PR description
- Update migration guides if needed
- Ensure database migrations are backward compatible
- Consider feature flags for gradual rollout

## Additional Resources

- See `docs/ARCHITECTURE.md` for detailed system architecture
- API documentation: https://getoutline.com/developers
- Run `yarn lint` before committing to catch issues early
