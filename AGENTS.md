# AGENTS.md - AI Agent Guidelines for Outline

## Overview

Outline is a fast, collaborative knowledge base built for teams using TypeScript, React, and Koa.

**Tech Stack:**
- **Frontend**: React + MobX + styled-components (compiled with Vite)
- **Backend**: Koa + Sequelize ORM + PostgreSQL + Redis
- **Real-time**: WebSocket collaboration using Y.js
- **Editor**: Prosemirror-based rich text editor

## Code Review Checklist

Before submitting changes, verify:

- [ ] All lint checks pass (`yarn lint`)
- [ ] All tests pass (`yarn test`)
- [ ] TypeScript compiles without errors (`yarn tsc`)
- [ ] New code follows existing patterns in the codebase
- [ ] No hardcoded secrets or credentials
- [ ] No use of `any` type - use proper types or generics
- [ ] Path aliases used instead of relative imports (`@server/*`, `@shared/*`, `~/*`)
- [ ] JSDoc comments added for public/exported functions

## Testing Requirements

- Write unit tests for new functionality
- Tests must be colocated with source files (e.g., `Component.test.tsx` next to `Component.tsx`)
- **Do not create new test directories** - tests belong next to their source files
- Maintain test coverage above thresholds:
  - Statements: 50%
  - Branches: 40%
  - Functions: 50%
  - Lines: 50%

### Running Tests

```bash
yarn test                    # Run all tests
yarn test path/to/file.test.ts  # Run specific test file (preferred)
yarn test:app                # Frontend tests (jsdom environment)
yarn test:server             # Backend tests (node environment)
yarn test:shared             # Shared code tests
```

## Code Style

- Use TypeScript for all new code
- Use functional React components with hooks (no class components)
- Use styled-components for CSS-in-JS styling
- Use `@action` decorators for MobX state mutations
- Event handlers should be prefixed with "handle" (e.g., `handleClick`)
- Always use curly braces for if statements
- Use strict equality (`===`) instead of loose equality
- No `console.log` in production code

### Import Aliases

| Alias       | Maps To      |
| ----------- | ------------ |
| `@server/*` | `./server/*` |
| `@shared/*` | `./shared/*` |
| `~/*`       | `./app/*`    |

## Pull Request Guidelines

- Keep PRs focused on a single change
- Include tests for new functionality
- Update documentation if behavior changes
- Ensure CI passes before requesting review
- Use conventional commit format for PR titles:
  - `feat: Add new feature`
  - `fix: Resolve bug in component`
  - `refactor: Improve code structure`
  - `docs: Update documentation`
  - `test: Add missing tests`

## Architecture Notes

### Directory Structure

| Directory | Purpose |
| --------- | ------- |
| `app/` | Frontend React application |
| `server/` | Backend Koa API server |
| `shared/` | Code shared between frontend and backend |
| `plugins/` | Plugin system for extending functionality |

### Key Backend Patterns

- **Routes**: `server/routes/api/` - API route handlers
- **Models**: `server/models/` - Sequelize database models
- **Commands**: `server/commands/` - Business logic commands
- **Policies**: `server/policies/` - Authorization logic (cancan)
- **Presenters**: `server/presenters/` - API response formatters

### Key Frontend Patterns

- **Components**: `app/components/` - Reusable UI components
- **Scenes**: `app/scenes/` - Full-page view components
- **Stores**: `app/stores/` - MobX stores for state management
- **Models**: `app/models/` - MobX observable state models

## Common Tasks

```bash
# Install dependencies
yarn install

# Start development server
yarn dev:watch

# Run linting
yarn lint

# Format code
yarn format

# Type check
yarn tsc

# Database migrations
yarn db:migrate           # Run migrations
yarn db:rollback          # Rollback last migration
yarn db:create-migration  # Create new migration
```

## Common Pitfalls to Avoid

1. **Don't mutate MobX observables directly** - Use `@action` decorators
2. **Don't use relative imports** - Use path aliases (`@server/*`, `@shared/*`, `~/*`)
3. **Don't import React unnecessarily** - JSX transform is enabled
4. **Don't skip error handling** - Always wrap async operations in try/catch
5. **Don't use `any` type** - Use proper types or generics
6. **Don't create test directories** - Colocate tests with source files

## Additional Resources

- See `CLAUDE.md` for comprehensive code style and patterns documentation
- See `docs/ARCHITECTURE.md` for detailed system architecture
- API documentation: https://getoutline.com/developers
