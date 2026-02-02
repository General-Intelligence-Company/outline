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

## Environment Setup

### Required Environment Variables

Create a `.env` file based on `.env.sample`. Key variables include:

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string | Yes |
| `REDIS_URL` | Redis connection string | Yes |
| `SECRET_KEY` | Secret for signing cookies/tokens | Yes |
| `UTILS_SECRET` | Additional secret for utilities | Yes |
| `URL` | Public URL of the application | Yes |
| `AWS_ACCESS_KEY_ID` | AWS credentials for S3 storage | For file uploads |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key | For file uploads |
| `AWS_S3_UPLOAD_BUCKET_NAME` | S3 bucket name | For file uploads |

### Docker Development

For local development with Docker:

```bash
# Start services (PostgreSQL, Redis)
docker-compose up -d

# Run database migrations
yarn db:migrate

# Start the development server
yarn dev
```

## Plugin Development

### Plugin Structure

Plugins are located in `plugins/` and follow this structure:

```
plugins/
  └── my-plugin/
      ├── client/           # React components
      │   └── index.tsx     # Client entry point
      ├── server/           # Koa routes and services
      │   └── index.ts      # Server entry point
      └── shared/           # Shared types and utilities
```

### Creating a New Plugin

1. Create a new directory in `plugins/`
2. Export a `PluginConfig` from `server/index.ts`:
   ```typescript
   import { PluginConfig } from "@server/types";

   export default {
     id: "my-plugin",
     name: "My Plugin",
     description: "What the plugin does",
     // Optional hooks
     hooks: {
       document: {
         beforeCreate: async (document) => { /* ... */ },
       },
     },
   } satisfies PluginConfig;
   ```

3. Register routes in the server entry point if needed
4. Export React components from `client/index.tsx` for UI integration

### Available Hooks

| Hook | Trigger |
|------|---------|
| `document.beforeCreate` | Before a document is created |
| `document.afterCreate` | After a document is created |
| `document.beforeUpdate` | Before a document is updated |
| `user.afterCreate` | After a user is created |

## Architecture Patterns

### Command Pattern

Commands encapsulate business logic and are located in `server/commands/`:

```typescript
// server/commands/documentCreator.ts
export default async function documentCreator({
  title,
  user,
  collection,
}: Props): Promise<Document> {
  // Validation
  // Business logic
  // Database operations
  // Event emission
  return document;
}
```

Use commands for:
- Operations that modify state
- Multi-step business processes
- Operations that need transaction handling

### Presenter Pattern

Presenters format data for API responses in `server/presenters/`:

```typescript
// server/presenters/document.ts
export default function presentDocument(
  document: Document,
  options?: Options
) {
  return {
    id: document.id,
    title: document.title,
    // ... only include fields the client needs
  };
}
```

Rules:
- Never expose internal fields (like hashed passwords)
- Keep response shapes consistent
- Handle null/undefined gracefully

### Policy Pattern (Authorization)

Policies define what actions users can perform, using a cancan-style approach:

```typescript
// server/policies/document.ts
allow(User, "read", Document, (user, document) => {
  return document.collection.memberships.some(
    m => m.userId === user.id
  );
});

allow(User, "update", Document, (user, document) => {
  return document.createdById === user.id ||
         user.isAdmin;
});
```

Check permissions in routes:
```typescript
authorize(user, "update", document);
```

## Debugging Tips

### Common Debug Commands

```bash
# View server logs with more detail
DEBUG=* yarn dev

# Run a specific test file
yarn test:server --testPathPattern="documentCreator"

# Check TypeScript errors in watch mode
yarn tsc --watch

# Inspect database state
yarn db:console
```

### Debugging WebSockets

For real-time collaboration issues:
1. Check browser DevTools → Network → WS tab
2. Look for Y.js sync messages
3. Verify Redis pub/sub is working

### Database Debugging

```bash
# Connect to database
yarn db:console

# View recent migrations
SELECT * FROM "SequelizeMeta" ORDER BY name DESC LIMIT 5;

# Check indexes
SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'documents';
```

## Common Pitfalls

### ❌ Avoid These Mistakes

1. **Don't use `any` or `unknown` types**
   ```typescript
   // Bad
   const data: any = fetchData();

   // Good
   interface DataShape { id: string; name: string; }
   const data: DataShape = fetchData();
   ```

2. **Don't forget transaction handling for multi-step operations**
   ```typescript
   // Bad - partial failure possible
   await document.save();
   await revision.save();

   // Good - atomic operation
   await sequelize.transaction(async (transaction) => {
     await document.save({ transaction });
     await revision.save({ transaction });
   });
   ```

3. **Don't hardcode URLs or environment-specific values**
   ```typescript
   // Bad
   const apiUrl = "https://app.getoutline.com/api";

   // Good
   const apiUrl = env.URL + "/api";
   ```

4. **Don't bypass the policy system for authorization**
   ```typescript
   // Bad - checking permissions manually
   if (user.id === document.createdById) { ... }

   // Good - use the policy system
   authorize(user, "update", document);
   ```

5. **Don't create database queries without indexes**
   - Always check if frequently-queried columns have indexes
   - Add indexes in migrations for new columns used in WHERE clauses

6. **Don't forget to handle async errors**
   ```typescript
   // Bad
   async function handler() {
     const result = await riskyOperation();
   }

   // Good
   async function handler() {
     try {
       const result = await riskyOperation();
     } catch (error) {
       Logger.error("Operation failed", error);
       throw new InternalError("Operation failed");
     }
   }
   ```

### ⚠️ Performance Gotchas

1. **N+1 Queries**: Always use `include` for related data
   ```typescript
   // Bad - N+1 queries
   const documents = await Document.findAll();
   for (const doc of documents) {
     const user = await doc.getCreatedBy();
   }

   // Good - single query with join
   const documents = await Document.findAll({
     include: [{ model: User, as: "createdBy" }],
   });
   ```

2. **Missing Pagination**: Always paginate list endpoints
   ```typescript
   const { limit = 25, offset = 0 } = ctx.request.query;
   const documents = await Document.findAll({ limit, offset });
   ```

3. **Unnecessary Re-renders**: Use React.memo and useMemo appropriately
   ```typescript
   const MemoizedComponent = React.memo(ExpensiveComponent);
   const computedValue = useMemo(() => expensiveCalculation(data), [data]);
   ```

## Migration Best Practices

### Creating Migrations

```bash
# Generate a new migration
yarn db:create-migration --name add-column-to-documents
```

### Migration Guidelines

1. **Always make migrations reversible**
   ```typescript
   export async function up(queryInterface) {
     await queryInterface.addColumn("documents", "archived", {
       type: DataTypes.BOOLEAN,
       defaultValue: false,
     });
   }

   export async function down(queryInterface) {
     await queryInterface.removeColumn("documents", "archived");
   }
   ```

2. **Test migrations locally before committing**
   ```bash
   yarn db:migrate        # Apply
   yarn db:migrate:undo   # Rollback
   yarn db:migrate        # Re-apply
   ```

3. **Handle data migrations carefully**
   - For large tables, batch updates to avoid locks
   - Consider running data migrations separately from schema changes

4. **Add indexes concurrently for large tables**
   ```typescript
   await queryInterface.sequelize.query(
     'CREATE INDEX CONCURRENTLY idx_documents_archived ON documents(archived)'
   );
   ```
