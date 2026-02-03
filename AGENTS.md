# Agents Guide - Outline Codebase

This document provides navigation and development guidance for AI agents and developers working with the Outline codebase.

## Navigation Guide

### Frontend Code (`/app`)

| Directory | Purpose |
|-----------|---------|
| `/app/components` | Reusable UI components |
| `/app/stores` | MobX state management stores |
| `/app/scenes` | Page-level components (routes) |
| `/app/hooks` | Custom React hooks |
| `/app/menus` | Menu components and context menus |
| `/app/models` | MobX data models |
| `/app/routes` | Route definitions |
| `/app/utils` | Frontend utility functions |
| `/app/actions` | Action creators |
| `/app/styles` | Global styles and themes |

### Backend Code (`/server`)

| Directory | Purpose |
|-----------|---------|
| `/server/routes` | API route handlers |
| `/server/models` | Sequelize database models |
| `/server/commands` | Business logic commands |
| `/server/policies` | Authorization policies |
| `/server/queues` | Background job processing |
| `/server/middlewares` | Koa middleware |
| `/server/presenters` | API response formatters |
| `/server/services` | External service integrations |
| `/server/migrations` | Database migrations |
| `/server/emails` | Email templates |
| `/server/collaboration` | Real-time collaboration logic |

### Shared Code (`/shared`)

| Directory | Purpose |
|-----------|---------|
| `/shared/types` | TypeScript type definitions |
| `/shared/utils` | Shared utility functions |
| `/shared/editor` | ProseMirror editor components |
| `/shared/i18n` | Internationalization (translations) |

### Plugins (`/plugins`)

Each plugin is self-contained with its own routes, models, etc.

| Category | Plugins |
|----------|---------|
| **Auth providers** | azure, google, oidc, slack, passkeys |
| **Integrations** | discord, github, linear, notion, figma, zapier, webhooks |
| **Analytics** | googleanalytics, matomo, umami |
| **Features** | email, diagrams, iframely, storage, enterprise |

## Testing Requirements

- **All new features must have tests**
- Tests are collocated with source files (`*.test.ts`)
- Do not create new test directories

### Test Environments

| Test Suite | Command | Environment |
|------------|---------|-------------|
| Frontend | `yarn test:app` | Jest with jsdom |
| Backend | `yarn test:server` | Jest with Node + PostgreSQL |
| Shared | `yarn test:shared` | Jest with Node and jsdom |

### Running Tests

```bash
# Run a specific test file (preferred)
yarn test path/to/file.test.ts

# Run frontend tests
yarn test:app

# Run backend tests (requires PostgreSQL)
yarn test:server

# Run shared code tests
yarn test:shared
```

## Code Review Checklist

1. **TypeScript types are properly defined**
   - No `any` types unless absolutely necessary
   - Use interfaces over type aliases for object shapes
   - Avoid type assertions (`as`, `!`)

2. **Linting passes**
   - Run `yarn lint` before committing
   - OxLint will auto-fix some issues via pre-commit hook

3. **Prettier formatting applied**
   - Run `yarn format` or let pre-commit hook handle it
   - printWidth: 80, trailingComma: es5

4. **Tests added for new functionality**
   - Collocate tests with source files
   - Mock external dependencies in `__mocks__` folder

5. **Translations added for user-facing strings**
   - Do not add translation strings manually
   - They are extracted automatically via pre-commit hook

6. **Migrations are reversible**
   - Include both `up` and `down` methods
   - Test rollback with `yarn db:rollback`

7. **Type checking passes**
   - Run `yarn tsc` to verify no type errors

## Common Pitfalls

### Database Changes
- Don't forget to run migrations when adding database changes
- Use `yarn db:create-migration` to generate migration files
- Always test rollback functionality

### MobX State Management
- Stores must use `@observable` and `@action` decorators
- Use `@computed` for derived values
- Keep business logic in stores, not components

### Path Aliases
- Use `@server/*` for server imports
- Use `@shared/*` for shared imports
- Use `~/*` for app (frontend) imports

### Pre-commit Hooks
- Husky runs automatically on commit
- Will auto-fix linting and formatting issues
- Extracts i18n translations automatically

### Backend Tests
- Require a running PostgreSQL instance
- CI uses PostgreSQL service container
- Set up local PostgreSQL for development testing

### React Patterns
- Use functional components with hooks
- Prefix event handlers with "handle" (e.g., `handleClick`)
- Use `React.memo`, `useMemo`, `useCallback` to prevent re-renders

## Plugin Development

### Plugin Structure

Plugins are located in `/plugins` directory. Each plugin can export:
- `server.ts` - Server-side functionality (routes, models, commands)
- `client.ts` - Client-side functionality (components, stores)

### Creating a Plugin

1. Create a new directory in `/plugins`
2. Export server functionality via `server.ts`
3. Export client functionality via `client.ts`
4. Register the plugin in the main plugin loader

### Existing Plugin Categories

- **Auth plugins**: Handle authentication providers
- **Integration plugins**: Connect with external services
- **Analytics plugins**: Track usage and metrics
- **Storage plugins**: Handle file storage (S3, local)

## Key Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| React | 17 | UI framework (not 18 yet) |
| Koa.js | - | HTTP server framework |
| Sequelize | - | PostgreSQL ORM |
| MobX | - | State management |
| ProseMirror | - | Rich text editing |
| Y.js | - | Real-time collaboration |
| styled-components | - | CSS-in-JS styling |
| Jest | 30 | Testing framework |
| Vite | - | Build tooling |

## CI/CD Pipeline

### GitHub Actions Workflow

Location: `.github/workflows/ci.yml`

### Pipeline Stages

```
setup → lint → types → changes → test/test-server → bundle-size
```

| Job | Description |
|-----|-------------|
| `setup` | Install dependencies with caching |
| `lint` | Run OxLint (`yarn lint --quiet`) |
| `types` | TypeScript type checking (`yarn tsc`) |
| `changes` | Detect which files changed (app, server, config) |
| `test` | Frontend tests (runs if app files changed) |
| `test-server` | Backend tests with PostgreSQL (4 shards, runs if server changed) |
| `bundle-size` | Bundle analysis with RelativeCI |

### CI Services

- **PostgreSQL 14.2**: Used for backend tests
- **Docker**: Multi-arch image builds (ARM64, AMD64)

### Other Workflows

| Workflow | Purpose |
|----------|---------|
| `docker.yml` | Build and publish Docker images on tags |
| `codeql-analysis.yml` | Security scanning |
| `stale.yml` | Auto-close stale issues/PRs |

## Quick Reference

### Essential Commands

```bash
# Development
yarn dev:watch          # Start dev server

# Quality Checks
yarn lint               # Run linter
yarn format             # Format code
yarn tsc                # Type check

# Testing
yarn test:app           # Frontend tests
yarn test:server        # Backend tests

# Database
yarn db:migrate         # Run migrations
yarn db:rollback        # Rollback migration
```

### File Patterns

- Source files: `*.ts`, `*.tsx`
- Test files: `*.test.ts` (collocated with source)
- Styles: styled-components (inline)
- Translations: Auto-extracted via i18next

### Import Aliases

```typescript
import { something } from "@server/models";   // Server code
import { something } from "@shared/utils";    // Shared code
import { something } from "~/components";     // App code
```
