# AI Agent Quick Reference

This document provides a streamlined quick reference for AI agents working on the Outline codebase. For comprehensive guidelines, see [CLAUDE.md](./CLAUDE.md).

## Quick Start

### Essential Commands

```bash
# Development
yarn install          # Install dependencies
yarn dev:watch        # Start dev server (frontend + backend)

# Quality checks (run before committing)
yarn lint             # Run oxlint
yarn tsc              # TypeScript type checking
yarn test             # Run all tests

# Testing
yarn test path/to/file.test.ts   # Run specific test
yarn test:app                     # Frontend tests (jsdom)
yarn test:server                  # Backend tests (node)
yarn test:shared                  # Shared code tests

# Database
yarn db:migrate           # Run migrations
yarn db:rollback          # Rollback last migration
yarn db:create-migration  # Create new migration
```

## Project Structure at a Glance

```
outline/
├── app/              # React frontend (MobX, styled-components)
│   ├── actions/      # Reusable actions
│   ├── components/   # UI components
│   ├── hooks/        # React hooks
│   ├── models/       # MobX models
│   ├── scenes/       # Page components
│   └── stores/       # MobX stores
├── server/           # Koa backend (Sequelize, PostgreSQL)
│   ├── commands/     # Business logic
│   ├── models/       # Database models
│   ├── policies/     # Authorization (cancan)
│   ├── presenters/   # API response formatters
│   ├── routes/api/   # API endpoints
│   └── test/         # Test helpers & factories
├── shared/           # Code shared between frontend/backend
│   ├── editor/       # Prosemirror editor
│   └── utils/        # Shared utilities
└── plugins/          # Plugin system
```

### Import Aliases

| Alias       | Maps To      | Example                              |
| ----------- | ------------ | ------------------------------------ |
| `@server/*` | `./server/*` | `import { User } from "@server/models/User"` |
| `@shared/*` | `./shared/*` | `import { formatDate } from "@shared/utils/date"` |
| `~/*`       | `./app/*`    | `import { Button } from "~/components/Button"` |

## Testing Patterns

### Test File Conventions

- **Location**: Tests are colocated with source files (e.g., `User.ts` → `User.test.ts`)
- **Do NOT create separate test directories**

### Jest Test Structure

```typescript
import { buildUser, buildTeam } from "@server/test/factories";

describe("feature name", () => {
  beforeAll(() => {
    // Setup that runs once before all tests
  });

  afterAll(() => {
    // Cleanup after all tests
  });

  describe("method or scenario", () => {
    it("should do expected behavior", async () => {
      // Arrange
      const user = await buildUser();

      // Act
      const result = await user.someMethod();

      // Assert
      expect(result).toBeDefined();
    });
  });
});
```

### Factory Functions

Use factory functions from `@server/test/factories` to create test data:

```typescript
import {
  buildUser,
  buildAdmin,
  buildViewer,
  buildTeam,
  buildCollection,
  buildDocument,
} from "@server/test/factories";

// Create with defaults
const user = await buildUser();

// Create with overrides
const admin = await buildAdmin({ teamId: team.id });

// Create related entities
const team = await buildTeam();
const collection = await buildCollection({ teamId: team.id });
```

### Coverage Requirements

| Metric     | Threshold |
| ---------- | --------- |
| Statements | 50%       |
| Branches   | 40%       |
| Functions  | 50%       |
| Lines      | 50%       |

## Code Review Checklist

Before submitting code, verify:

### TypeScript
- [ ] No `any` type usage
- [ ] Use `import type` for type-only imports
- [ ] No unnecessary type assertions (`as`, `!`)
- [ ] Prefer `interface` over `type` for objects

### React/Frontend
- [ ] Functional components with hooks
- [ ] Event handlers prefixed with `handle` (e.g., `handleClick`)
- [ ] Use styled-components for styling
- [ ] No unnecessary `import React`
- [ ] Self-closing tags for empty elements

### Backend
- [ ] Use validation middleware for requests
- [ ] Format responses through presenters
- [ ] Check authorization via policies
- [ ] Handle errors with proper error types

### Tests
- [ ] Tests colocated with source files
- [ ] Use factory functions for test data
- [ ] Coverage thresholds maintained

## Common Pitfalls to Avoid

| Pitfall | Wrong | Correct |
| ------- | ----- | ------- |
| Relative imports | `../../../server/models/User` | `@server/models/User` |
| React import | `import React from "react"` | *(not needed)* |
| Loose equality | `value == null` | `value === null \|\| value === undefined` |
| Missing braces | `if (x) return y;` | `if (x) { return y; }` |
| Direct MobX mutation | `user.name = "new"` | Use `@action` decorator |
| Missing JSDoc | `export function fn()` | Add JSDoc comment |
| Non-self-closing | `<div></div>` | `<div />` |

## CI/CD Pipeline

The CI pipeline runs automatically on PRs to `main`:

1. **Setup** - Install dependencies, cache node_modules
2. **Lint** - Run oxlint (`yarn lint --quiet`)
3. **Types** - TypeScript checking (`yarn tsc`)
4. **Tests** - Run Jest tests (sharded for server tests)
   - Frontend: `yarn test:app`
   - Server: Sharded across 4 runners with PostgreSQL
5. **Bundle Size** - Track bundle size changes (RelativeCI)

### Pre-commit Hooks

Husky + lint-staged runs automatically on commit:
- Linting with oxlint
- Format checking

## PR Guidelines

### Title Format

Use conventional commits:
- `feat: Add new feature`
- `fix: Resolve bug in component`
- `refactor: Improve code structure`
- `docs: Update documentation`
- `test: Add missing tests`
- `chore: Update dependencies`

### Description Template

```markdown
## Summary
Brief description of changes

## Motivation
Why this change is needed

## Testing
How the changes were tested

## Screenshots (if applicable)
For UI changes
```

## Additional Resources

- **Detailed Guidelines**: [CLAUDE.md](./CLAUDE.md)
- **API Docs**: https://getoutline.com/developers
- **Architecture**: `docs/ARCHITECTURE.md`
