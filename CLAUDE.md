# Outline - Collaborative Knowledge Base

## Project Overview

Outline is a fast, collaborative knowledge base built for teams. It's built with React and TypeScript in both frontend and backend, uses a real-time collaboration engine, and is designed for excellent performance and user experience.

- **Fullstack monorepo**: React frontend + Koa.js backend
- **Database**: PostgreSQL with Sequelize ORM
- **Caching/Queues**: Redis
- **Real-time collaboration**: WebSockets and Y.js
- **Rich text editor**: Built on ProseMirror
- **Deployment**: Self-hosted or cloud service

There is a web client which is fully responsive and works on mobile devices.

## Architecture

```
/workspace/repo/
├── app/              # React 17 frontend with MobX state management, styled-components
├── server/           # Koa.js backend with RESTful API routes
├── shared/           # Shared code (types, utils, i18n, editor components)
├── plugins/          # 20 plugins for auth, integrations, storage, analytics
├── public/           # Static assets served directly
└── docs/             # Documentation (see /docs/ARCHITECTURE.md for details)
```

### Frontend (`/app`)
- React 17 with MobX for state management
- styled-components for CSS-in-JS
- Vite for build tooling

### Backend (`/server`)
- Koa.js HTTP server
- Sequelize ORM with PostgreSQL
- Background job processing with queues
- RESTful API design

### Shared (`/shared`)
- TypeScript types and interfaces
- Utility functions
- i18n translations
- Editor components (ProseMirror)

### Plugins (`/plugins`)
- **Auth**: azure, google, oidc, slack, passkeys
- **Integrations**: discord, github, linear, notion, figma, zapier, webhooks
- **Analytics**: googleanalytics, matomo, umami
- **Features**: email, diagrams, iframely, storage, enterprise

## Development Commands

```bash
# Development
yarn dev:watch          # Start dev server (backend + frontend)

# Building
yarn build              # Production build

# Linting & Formatting
yarn lint               # Run OxLint
yarn format             # Run Prettier

# Type Checking
yarn tsc                # TypeScript type check

# Testing
yarn test               # Run all tests
yarn test:app           # Frontend tests only
yarn test:server        # Backend tests only (requires PostgreSQL)
yarn test:shared        # Shared code tests

# Database
yarn db:migrate         # Run migrations
yarn db:rollback        # Rollback last migration
yarn db:create-migration # Create new migration
yarn db:reset           # Drop, create, and migrate
```

## Code Style & Patterns

- **TypeScript strict mode**: strictNullChecks, noImplicitAny enabled
- **Linter**: OxLint (Rust-based, fast)
- **Formatter**: Prettier (printWidth: 80, trailingComma: es5)
- **Path aliases**: `@server/*`, `@shared/*`, `~/*` (app)

## Key Patterns

- **State Management**: MobX stores in frontend (`app/stores/`)
- **Database**: Sequelize models with migrations (`server/models/`)
- **Authorization**: Policy-based in server (`server/policies/`)
- **Business Logic**: Command pattern (`server/commands/`)
- **Extensibility**: Plugin architecture (`plugins/`)

## Pre-commit Hooks

Husky + lint-staged automatically runs on commit:
- Prettier formatting on staged files
- OxLint with `--fix`
- i18n translation extraction
- Yarn package deduplication

## Environment Setup

- Requires **Node.js 20+**
- Requires **PostgreSQL** and **Redis**
- Use **yarn** (v4.11.0 Berry) as package manager
- Copy `.env.sample` to `.env` for local development

---

## Instructions

You're an expert in the following areas:

- TypeScript
- React and React Router
- MobX and MobX-React
- Node.js and Koa
- Sequelize ORM
- PostgreSQL
- Redis
- HTML, CSS and Styled Components
- Prosemirror (rich text editor)
- WebSockets and real-time collaboration

## General Guidelines

- Critical – Do not create new markdown (.md) files.
- Use early returns for readability.
- Emphasize type safety and static analysis.
- Follow consistent Prettier formatting.
- Do not replace smart quotes ("") or ('') with simple quotes ("").
- Do not add translation strings manually; they will be extracted automatically from the codebase.

## Dependencies and Upgrading

- Use yarn for all dependency management.
- After updating dependency versions, install to update lockfiles:

```bash
yarn install
```

## TypeScript Usage

- Use strict mode.
- Avoid "unknown" unless absolutely necessary.
- Never use "any".
- Prefer type definitions; avoid type assertions (as, !).
- Always use curly braces for if statements.
- Avoid # for private properties.
- Prefer interface over type for object shapes.

## Classes & Code Organization

### Class Member Order

1. Public static variables
2. Public static methods
3. Public variables
4. Public methods
5. Protected variables & methods
6. Private variables & methods

### Exports

- Exported members must appear at the top of the file.
- Prefer named exports for components & classes.
- Document ALL public/exported functions with JSDoc.

## React Usage

- Use functional components with hooks.
- Event handlers should be prefixed with "handle", like "handleClick" for onClick.
- Avoid unnecessary re-renders by using React.memo, useMemo, and useCallback appropriately.
- Use descriptive prop types with TypeScript interfaces.
- Do not import React unless it is used directly.
- Use styled-components for component styling.
- Ensure high accessibility (a11y) standards using ARIA roles and semantic HTML.

## MobX State Management

- Use MobX stores for global state management.
- Keep stores in `app/stores/`.
- Use `observable`, `action`, and `computed` decorators appropriately.
- Prefer computed values over manual calculations in render.
- Keep business logic in stores, not components.

## Database & ORM

- Use Sequelize models in `server/models/`.
- Generate migrations with Sequelize CLI:

```bash
yarn sequelize migration:create --name=add-field-to-table
```

- Run migrations with `yarn db:migrate`.
- Use transactions for multi-table operations.
- Add appropriate indexes for query performance.
- Always handle database errors gracefully.

## API Design

- RESTful endpoints under `/api/`.
- Authentication endpoints under `/auth/`.
- Use consistent error responses.
- Validate request data using the validation middleware and schemas
- Use presenters to format API responses.
- Keep API routes thin, use model methods for business logic, or commands if logic spans multiple models.

## Authentication & Authorization

- JWT tokens for authentication.
- Policies in `server/policies/` for authorization.
- Use cancan-style ability checks.
- Use authenticated middleware for protected routes.
- Always verify user permissions before data access.

## Real-time Collaboration

- WebSocket connections for real-time updates.
- Use Y.js for collaborative editing.
- Handle connection state changes gracefully.

## Documentation

- All public/exported functions & classes must have JSDoc.
- Include:
  - Description
  - @param and @return (start lowercase, end with period)
  - @throws if applicable
- Add a newline between the description and the @ block.
- Use correct punctuation.

## Testing

- Run tests with Jest:

```bash
# Run a specific test file (preferred)
yarn test path/to/test.spec.ts

# Run every test (avoid)
yarn test

# Run test suites (avoid)
yarn test:app      # All frontend tests
yarn test:server   # All backend tests
yarn test:shared   # All shared code tests
```

- Write unit tests for utilities and business logic in a collocated .test.ts file.
- Do not create new test directories
- Mock external dependencies appropriately in **mocks** folder.
- Aim for high code coverage but focus on critical paths.

## Code Quality

- Use Oxlint for linting: `yarn lint`
- Format code with Prettier: `yarn format`
- Check types with TypeScript: `yarn tsc`
- Pre-commit hooks run automatically via Husky.
- Fix linting issues before committing.

## Error Handling

- Use custom error classes in `server/errors.ts`.
- Always catch and handle errors appropriately.
- Log errors with appropriate context.
- Return user-friendly error messages.
- Never expose sensitive information in errors.

## Performance

- Use React.memo for expensive components.
- Implement pagination for large lists.
- Use database indexes effectively.
- Cache expensive computations.
- Monitor performance with appropriate tools.
- Lazy load routes and components where appropriate.

## Security

- Sanitize all user input.
- Use CSRF protection.
- Use rateLimiter middleware for sensitive endpoints.
- Follow OWASP guidelines.
- Never store sensitive data in plain text.
- Use environment variables for secrets.
