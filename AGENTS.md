# AI Agent Guidelines for Outline

Outline is a fast, collaborative knowledge base built for teams. It's built with React and TypeScript in both frontend and backend, uses a real-time collaboration engine, and is designed for excellent performance and user experience. The backend is a Koa server with an RPC API and uses PostgreSQL and Redis. The application can be self-hosted or used as a cloud service.

There is a web client which is fully responsive and works on mobile devices.

**Monorepo Structure:**

- **`app/`** - React web application with MobX state management
- **`server/`** - Koa API server with Sequelize ORM and background workers
- **`shared/`** - Shared TypeScript types, utilities, and editor components
- **`plugins/`** - Plugin system for extending functionality
- **`public/`** - Static assets served directly
- **Various config files** - TypeScript, Vite, Jest, Prettier, Oxlint configurations

Refer to /docs/ARCHITECTURE.md for detailed architecture documentation.

---

## Quick Reference

### Key Commands

```bash
# Development
yarn dev:watch              # Start full dev environment (backend + frontend)
yarn dev:backend            # Backend only with hot reload
yarn vite:dev               # Frontend only (Vite dev server)

# Testing
yarn test path/to/file.test.ts  # Run specific test (preferred)
yarn test:app                   # Frontend tests only
yarn test:server                # Backend tests only
yarn test:shared                # Shared code tests

# Code Quality
yarn lint                   # Run Oxlint
yarn format                 # Run Prettier
yarn tsc                    # TypeScript type checking

# Database
yarn db:migrate             # Run migrations
yarn db:rollback            # Rollback last migration
yarn db:create-migration    # Create new migration

# Build
yarn build                  # Full production build
```

### Navigation Guide

| What you're looking for | Where to find it |
|-------------------------|------------------|
| React components | `app/components/` |
| Full-page views | `app/scenes/` |
| Frontend state/stores | `app/stores/` |
| Frontend models | `app/models/` |
| API routes | `server/routes/api/` |
| Database models | `server/models/` |
| Authorization policies | `server/policies/` |
| Database migrations | `server/migrations/` |
| Shared utilities | `shared/utils/` |
| Editor implementation | `shared/editor/` |
| Plugin implementations | `plugins/<name>/` |
| Test factories | `server/test/factories.ts` |
| Custom errors | `server/errors.ts` |
| Email templates | `server/emails/templates/` |

### TypeScript Path Aliases

```typescript
import { User } from "@server/models/User";     // server/*
import { formatDate } from "@shared/utils/date"; // shared/*
import { useStores } from "~/stores";            // app/*
```

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

### Test Organization

Tests are **collocated** with source files (`.test.ts` next to the source file).

```bash
# Always run specific tests, not the entire suite
yarn test path/to/feature.test.ts

# Example: Testing a model
yarn test server/models/User.test.ts
```

### Jest Configuration

Four test projects are configured:
- `server` – Backend tests (Node environment)
- `app` – Frontend tests (jsdom environment)
- `shared-node` – Shared code in Node
- `shared-jsdom` – Shared code in browser

### Writing Tests

```typescript
// Use factories for test data
import { buildUser, buildDocument } from "@server/test/factories";

describe("Feature", () => {
  it("should do something", async () => {
    const user = await buildUser();
    const document = await buildDocument({ userId: user.id });
    // Test logic here
  });
});
```

- Write unit tests for utilities and business logic in a collocated .test.ts file.
- Do not create new test directories.
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

---

## Code Review Checklist

### TypeScript

- [ ] No `any` types used (use proper types or `unknown` with type guards)
- [ ] No type assertions (`as`, `!`) unless absolutely necessary
- [ ] Interfaces used for object shapes (not `type`)
- [ ] Strict mode compliance

### React Components

- [ ] Functional components with hooks
- [ ] Event handlers prefixed with `handle` (e.g., `handleClick`)
- [ ] Props defined with TypeScript interfaces
- [ ] Styled-components used for styling
- [ ] Accessibility attributes included (ARIA roles, semantic HTML)
- [ ] React.memo, useMemo, useCallback used where appropriate

### Backend Code

- [ ] API routes are thin (business logic in models/commands)
- [ ] Policies used for authorization checks
- [ ] Presenters used for API responses
- [ ] Transactions used for multi-model operations
- [ ] Custom error classes used from `server/errors.ts`

### Documentation

- [ ] JSDoc for all public/exported functions
- [ ] Description, @param, @return included
- [ ] @throws documented if applicable

### General

- [ ] No `console.log` or debug statements
- [ ] No hardcoded secrets or credentials
- [ ] Smart quotes preserved (not converted to simple quotes)
- [ ] Early returns used for readability
- [ ] Curly braces used for all if statements

---

## Common Pitfalls

### Avoid These Mistakes

1. **Creating markdown files**: Never create new `.md` files unless explicitly requested
2. **Manual translation strings**: Translations are auto-extracted; don't add manually
3. **Using `any`**: Always use proper types
4. **Forgetting transactions**: Multi-model operations need transactions
5. **Skipping authorization**: Always check policies before data access
6. **Large test suites**: Run specific tests, not `yarn test` for everything
7. **Ignoring type errors**: Fix all TypeScript errors before committing

### Common Patterns

#### MobX Store Pattern

```typescript
// app/stores/ExampleStore.ts
import { observable, action, computed } from "mobx";
import RootStore from "./RootStore";

export default class ExampleStore {
  @observable data: Item[] = [];

  constructor(rootStore: RootStore) {
    this.rootStore = rootStore;
  }

  @action
  addItem = (item: Item) => {
    this.data.push(item);
  };

  @computed
  get itemCount() {
    return this.data.length;
  }
}
```

#### API Route Pattern

```typescript
// server/routes/api/example.ts
router.post("example.create", auth(), async (ctx) => {
  const { name } = ctx.request.body;
  const { user } = ctx.state.auth;

  authorize(user, "create", Example);

  const example = await Example.create({ name, userId: user.id });

  ctx.body = {
    data: presentExample(example),
  };
});
```

#### Policy Pattern

```typescript
// server/policies/example.ts
allow(User, "read", Example, (user, example) =>
  user.teamId === example.teamId
);

allow(User, "update", Example, (user, example) =>
  example.userId === user.id
);
```

---

## Key Dependencies

| Category | Package | Purpose |
|----------|---------|---------|
| Frontend | React 17 | UI framework |
| State | MobX 4 | State management |
| Routing | React Router 5 | Client-side routing |
| Styling | Styled Components 5 | Component styling |
| Backend | Koa 3 | HTTP server |
| ORM | Sequelize 6 | Database ORM |
| Queue | Bull | Job queue |
| WebSocket | Socket.io 4 | Real-time updates |
| Editor | Prosemirror | Rich text editor |
| Collaboration | Y.js | CRDT-based sync |
| Database | PostgreSQL | Primary database |
| Cache | Redis | Caching and queues |

---

## Example Workflows

### Adding a New Feature

1. **Plan the feature**
   - Identify affected components (frontend, backend, shared)
   - Check existing patterns in similar features

2. **Backend changes** (if needed)
   - Create/modify models in `server/models/`
   - Add migration: `yarn db:create-migration --name=feature-name`
   - Add API routes in `server/routes/api/`
   - Create policies in `server/policies/`
   - Add presenter in `server/presenters/`

3. **Frontend changes**
   - Add/modify stores in `app/stores/`
   - Add/modify models in `app/models/`
   - Create components in `app/components/`
   - Create scenes in `app/scenes/`
   - Add routes in `app/routes/`

4. **Testing**
   - Add tests collocated with source files
   - Run `yarn test path/to/feature.test.ts`

5. **Quality checks**
   - `yarn lint` – Check linting
   - `yarn tsc` – Check types
   - `yarn format` – Format code

### Fixing a Bug

1. **Reproduce the issue**
   - Understand the expected vs actual behavior
   - Identify the affected code path

2. **Locate the bug**
   - Use grep/search to find relevant files
   - Check related tests for context

3. **Fix the issue**
   - Make minimal, focused changes
   - Preserve existing patterns and styles

4. **Add regression test**
   - Write a test that would have caught the bug
   - Place test file next to the source file

5. **Verify the fix**
   - Run affected tests
   - Run lint and type checks

### Adding an API Endpoint

1. **Create the route** in `server/routes/api/<resource>.ts`

2. **Add validation** using the validation middleware

3. **Check authorization** with policies

4. **Implement business logic** in model methods or commands

5. **Format response** with a presenter

6. **Add tests** in a collocated test file

### Modifying the Database Schema

1. **Create migration**
   ```bash
   yarn db:create-migration --name=describe-change
   ```

2. **Edit migration file** in `server/migrations/`

3. **Update model** in `server/models/`

4. **Run migration**
   ```bash
   yarn db:migrate
   ```

5. **Test thoroughly** – schema changes can have wide impact

---

## Backend Services

| Service | Description | Required |
|---------|-------------|----------|
| `web` | Main API server | Yes |
| `worker` | Background job processor | Yes |
| `websockets` | Real-time updates | No |
| `collaboration` | Document collaboration | No |
| `cron` | Scheduled tasks | No |
| `admin` | Queue debugging UI | Dev only |

---

## Plugin Development

Plugins extend Outline's functionality. Each plugin has:

```
plugins/<name>/
├── plugin.json      # Plugin metadata
├── client/          # Frontend components
├── server/          # Backend routes/logic
└── shared/          # Shared code
```

Example `plugin.json`:
```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "description": "Plugin description",
  "priority": 100
}
```

---

## Debugging Tips

### Frontend Debugging

- Use React DevTools for component inspection
- Use MobX DevTools for state debugging
- Check browser console for errors
- Use `yarn vite:dev` for hot reload during development

### Backend Debugging

- Use `--inspect=0.0.0.0` flag for Node.js debugging
- Check `server/errors.ts` for custom error types
- Use Winston logger for structured logging
- Check Redis and PostgreSQL connections if services fail

### Common Issues

| Issue | Solution |
|-------|----------|
| Database connection fails | Check `.env` for correct `DATABASE_URL` |
| Redis connection fails | Ensure Redis is running, check `REDIS_URL` |
| Type errors | Run `yarn tsc` to identify issues |
| Tests fail | Check test setup files and mock configurations |
| Build fails | Clear `build/` directory and try again |

---

## Additional Resources

- **Architecture docs**: `/docs/ARCHITECTURE.md`
- **Services docs**: `/docs/SERVICES.md`
- **API documentation**: https://getoutline.com/developers
- **Translation guide**: `/docs/TRANSLATION.md`
- **Security policy**: `/docs/SECURITY.md`
