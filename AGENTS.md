# AI Agent Guidelines for Outline

This document provides guidance for AI agents working on the Outline codebase. Outline is a fast, collaborative knowledge base built for teams.

## Project Overview

Outline is a TypeScript monorepo containing:

- **Frontend**: React web application with MobX state management
- **Backend**: Koa API server with Sequelize ORM, PostgreSQL, and Redis
- **Real-time**: WebSocket-based collaboration using Y.js
- **Editor**: Prosemirror-based rich text editor

## Directory Structure

### Key Directories

| Directory  | Purpose                                                     |
| ---------- | ----------------------------------------------------------- |
| `app/`     | React web application (components, stores, scenes, hooks)   |
| `server/`  | Koa API server (routes, models, commands, policies, queues) |
| `shared/`  | Shared TypeScript types, utilities, and editor components   |
| `plugins/` | Plugin system for extending functionality                   |
| `public/`  | Static assets served directly                               |
| `docs/`    | Architecture and development documentation                  |

### Frontend (`app/`)

- `app/components/` - Reusable UI components
- `app/scenes/` - Page-level components (routes)
- `app/stores/` - MobX stores for state management
- `app/hooks/` - Custom React hooks
- `app/models/` - Client-side data models
- `app/utils/` - Frontend utilities

### Backend (`server/`)

- `server/routes/` - API route handlers
- `server/models/` - Sequelize database models
- `server/commands/` - Business logic commands
- `server/policies/` - Authorization policies
- `server/queues/` - Background job processors
- `server/middlewares/` - Koa middleware
- `server/presenters/` - API response formatters
- `server/migrations/` - Database migrations

### Shared (`shared/`)

- `shared/types.ts` - Common TypeScript types
- `shared/editor/` - Prosemirror editor components
- `shared/utils/` - Shared utility functions
- `shared/validations.ts` - Validation schemas

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

# Run specific test file
yarn test path/to/file.test.ts

# Run test suites
yarn test:app      # Frontend tests
yarn test:server   # Backend tests
yarn test:shared   # Shared code tests

# Database migrations
yarn db:migrate           # Run migrations
yarn db:rollback          # Rollback last migration
yarn db:create-migration  # Create new migration
```

## Import Conventions

Use path aliases instead of relative imports:

```typescript
// ✓ Correct
import { User } from "@server/models/User";
import { formatDate } from "@shared/utils/date";
import { Button } from "~/components/Button";

// ✗ Incorrect - avoid deep relative imports
import { User } from "../../../server/models/User";
```

### Path Aliases

| Alias       | Maps To      |
| ----------- | ------------ |
| `@server/*` | `./server/*` |
| `@shared/*` | `./shared/*` |
| `~/*`       | `./app/*`    |

## Testing Requirements

### Test File Location

Tests are colocated with source files using the `.test.ts` or `.test.tsx` extension:

```
server/models/User.ts
server/models/User.test.ts
```

### Running Tests

```bash
# Run a specific test file (preferred)
yarn test path/to/file.test.ts

# Run all tests
yarn test
```

### Writing Tests

- Use Jest for all tests
- Mock external dependencies in `__mocks__/` folders
- Focus on critical paths and business logic
- Do not create new test directories

## Code Review Checklist

When reviewing or writing code, verify:

### TypeScript

- [ ] No use of `any` type
- [ ] Avoid `unknown` unless necessary
- [ ] Prefer `interface` over `type` for object shapes
- [ ] Avoid type assertions (`as`, `!`)
- [ ] Strict null checks are respected

### React Components

- [ ] Use functional components with hooks
- [ ] Event handlers prefixed with "handle" (e.g., `handleClick`)
- [ ] Use styled-components for styling
- [ ] Ensure accessibility (ARIA roles, semantic HTML)
- [ ] Avoid unnecessary re-renders (use `React.memo`, `useMemo`, `useCallback`)

### Code Style

- [ ] Use early returns for readability
- [ ] Always use curly braces for if statements
- [ ] Named exports for components and classes
- [ ] JSDoc for all public/exported functions
- [ ] No console.log in production code

### API Routes

- [ ] Validate request data with validation middleware
- [ ] Use presenters for response formatting
- [ ] Check user authorization via policies
- [ ] Handle errors gracefully

## Common Pitfalls to Avoid

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

## Key Dependencies

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

## Additional Resources

- See `CLAUDE.md` for detailed coding standards
- See `docs/ARCHITECTURE.md` for system architecture
- Run `yarn lint` before committing to catch issues early
