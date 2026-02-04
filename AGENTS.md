# AI Agent Guidelines for Outline

This document provides comprehensive guidelines for AI agents working on the Outline codebase. It complements `CLAUDE.md` with additional context, detailed checklists, and practical examples for code review and development.

## Project Context

### What is Outline?

Outline is a fast, collaborative knowledge base built for teams. It enables organizations to create, share, and manage documentation with real-time collaboration features.

**Key Features:**
- Real-time collaborative editing
- Rich text editor with markdown support
- Nested document collections
- Full-text search
- Multiple authentication providers (Slack, Google, Azure, Discord, OIDC)
- Extensible plugin architecture

### Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Frontend** | React 17, MobX 4, styled-components | UI framework with reactive state management |
| **Backend** | Koa 3, Node.js 22 | REST API server |
| **Database** | PostgreSQL, Sequelize 6 | Data persistence and ORM |
| **Cache/Queue** | Redis, ioredis, Bull | Caching and background job processing |
| **Real-time** | Socket.io, Y.js, Hocuspocus | WebSocket-based collaboration |
| **Editor** | ProseMirror | Rich text editing |
| **Build** | Vite/Rolldown | Frontend bundling |
| **Testing** | Jest | Unit and integration testing |
| **Linting** | oxlint | Code quality |

### Monorepo Structure

```
outline/
├── app/              # Frontend React application
│   ├── actions/      # Reusable actions (navigate, open, create)
│   ├── components/   # Reusable UI components
│   ├── editor/       # Editor-specific React components
│   ├── hooks/        # Custom React hooks
│   ├── menus/        # Context menus
│   ├── models/       # MobX observable state models
│   ├── routes/       # Route definitions (lazy-loaded)
│   ├── scenes/       # Full-page view components
│   ├── stores/       # Collections of models and fetch logic
│   ├── types/        # TypeScript types
│   └── utils/        # Frontend utilities
├── server/           # Backend Koa API server
│   ├── routes/       # API route handlers
│   │   ├── api/      # REST API endpoints
│   │   └── auth/     # Authentication routes
│   ├── commands/     # Business logic commands
│   ├── models/       # Sequelize database models
│   ├── policies/     # Authorization logic (cancan)
│   ├── presenters/   # API response formatters
│   ├── middlewares/  # Koa middleware
│   ├── migrations/   # Database migrations
│   ├── queues/       # Bull job queue definitions
│   └── utils/        # Backend utilities
├── shared/           # Code shared between frontend and backend
│   ├── editor/       # ProseMirror editor components
│   ├── i18n/         # Internationalization
│   ├── styles/       # Global styles and colors
│   └── utils/        # Shared utility methods
├── plugins/          # Plugin system for extending functionality
└── docs/             # Documentation
```

### Path Aliases

Always use path aliases instead of relative imports:

| Alias | Maps To | Example |
|-------|---------|---------|
| `@server/*` | `./server/*` | `import { User } from "@server/models/User"` |
| `@shared/*` | `./shared/*` | `import { formatDate } from "@shared/utils/date"` |
| `~/*` | `./app/*` | `import { Button } from "~/components/Button"` |

---

## Code Review Checklist for AI Agents

Use this checklist when reviewing or writing code for the Outline codebase.

### TypeScript Patterns

- [ ] **No `any` type** - Use proper types or generics
- [ ] **Avoid `unknown`** - Only use when absolutely necessary
- [ ] **Prefer `interface`** - Use `interface` over `type` for object shapes
- [ ] **Minimize type assertions** - Avoid `as` and `!` operators
- [ ] **Strict null checks** - Always handle null/undefined cases
- [ ] **Consistent type imports** - Use `import type` for type-only imports

```typescript
// Correct
import type { User } from "@server/models/User";
import { UserModel } from "@server/models/User";

// Incorrect
import { User } from "@server/models/User"; // when only using as a type
```

### React Component Patterns

- [ ] **Functional components only** - No class components
- [ ] **Event handler naming** - Prefix with "handle" (e.g., `handleClick`, `handleSubmit`)
- [ ] **styled-components** - Use for all component styling
- [ ] **No React import** - JSX transform is enabled
- [ ] **Performance optimization** - Use `React.memo`, `useMemo`, `useCallback` appropriately
- [ ] **Self-closing tags** - Use for empty elements (`<div />` not `<div></div>`)
- [ ] **Accessibility** - Include ARIA roles and semantic HTML

```tsx
// Correct - functional component with styled-components
const StyledButton = styled.button`
  background: ${(props) => props.theme.primary};
`;

function MyButton({ onClick, children }: Props) {
  const handleClick = useCallback(() => {
    onClick?.();
  }, [onClick]);

  return <StyledButton onClick={handleClick}>{children}</StyledButton>;
}

export default observer(MyButton);
```

### MobX State Management

- [ ] **Use `@action` decorators** - For all state mutations
- [ ] **Use `@computed`** - For derived state
- [ ] **Wrap components with `observer`** - For reactive rendering
- [ ] **Keep stores focused** - Single responsibility principle

```typescript
// Correct - using MobX actions
class UserStore {
  @observable name = "";

  @action
  updateName(name: string) {
    this.name = name;
  }

  @computed
  get displayName() {
    return this.name || "Anonymous";
  }
}
```

### Koa Backend Patterns

- [ ] **Request validation** - Use validation middleware for all request data
- [ ] **Response formatting** - Always use presenters for API responses
- [ ] **Authorization** - Check permissions via cancan policies
- [ ] **Error handling** - Use proper error types, never swallow errors
- [ ] **Commands pattern** - Use commands for complex business logic

```typescript
// Correct API route pattern
router.post(
  "documents.create",
  auth(),
  validate(T.DocumentsCreateSchema),
  async (ctx) => {
    const { title, collectionId } = ctx.input.body;
    const { user } = ctx.state.auth;

    authorize(user, "createDocument", collection);

    const document = await documentCreator({
      title,
      collectionId,
      user,
      ip: ctx.request.ip,
    });

    ctx.body = {
      data: presentDocument(document),
    };
  }
);
```

### Database Patterns (Sequelize)

- [ ] **Migrations** - Always create migrations for schema changes
- [ ] **Rollback support** - Include down migration
- [ ] **Backward compatible** - Migrations must not break existing data
- [ ] **Indexes** - Add indexes for frequently queried columns
- [ ] **Foreign keys** - Properly define constraints
- [ ] **Transactions** - Use transactions for multi-model operations

```typescript
// Correct migration pattern
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.addColumn("documents", "archived_at", {
      type: Sequelize.DATE,
      allowNull: true,
    });
    await queryInterface.addIndex("documents", ["archived_at"]);
  },

  async down(queryInterface) {
    await queryInterface.removeIndex("documents", ["archived_at"]);
    await queryInterface.removeColumn("documents", "archived_at");
  },
};
```

---

## Testing Requirements

### Coverage Thresholds

| Metric | Threshold |
|--------|-----------|
| Statements | 50% |
| Branches | 40% |
| Functions | 50% |
| Lines | 50% |

### Test File Conventions

- **Location**: Tests are colocated with source files
- **Naming**: Use `.test.ts` or `.test.tsx` extension
- **Framework**: Jest for all tests

```
server/models/User.ts
server/models/User.test.ts    # Colocated test file
```

### Running Tests

```bash
# Run specific test file (preferred)
yarn test path/to/file.test.ts

# Run all tests
yarn test

# Run by environment
yarn test:app      # Frontend tests (jsdom)
yarn test:server   # Backend tests (node, requires PostgreSQL)
yarn test:shared   # Shared code tests (both environments)
```

### What Needs Testing

| Change Type | Testing Requirements |
|-------------|---------------------|
| New API endpoint | Unit tests for handler, integration tests for full flow |
| New React component | Component rendering tests, interaction tests |
| Business logic | Unit tests for commands/utilities |
| Database model | Model validation tests, association tests |
| Bug fix | Regression test proving the fix |

### Test Patterns

```typescript
// Backend test example
describe("User model", () => {
  it("should hash password on create", async () => {
    const user = await buildUser({ password: "plaintext" });
    expect(user.passwordHash).not.toBe("plaintext");
  });
});

// Frontend test example
describe("Button component", () => {
  it("should call onClick when clicked", () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    fireEvent.click(screen.getByText("Click me"));
    expect(handleClick).toHaveBeenCalled();
  });
});
```

---

## PR Guidelines

### Commit Message Format

Use conventional commit format:

```
<type>: <description>

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code refactoring (no functional change)
- `docs`: Documentation only
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples:**
```
feat: Add document archiving functionality
fix: Resolve infinite loop in search component
refactor: Extract document presenter logic
docs: Update API documentation for collections
test: Add tests for user authentication flow
chore: Update dependencies to latest versions
```

### PR Description Template

```markdown
## Summary
Brief description of the changes and their purpose.

## Motivation
Why this change is needed - the problem it solves or feature it adds.

## Testing
How the changes were tested:
- [ ] Unit tests added/updated
- [ ] Manual testing performed
- [ ] Integration tests pass

## Screenshots
(For UI changes - include before/after screenshots)

## Breaking Changes
(Note any breaking changes and migration steps)
```

### Required Checks Before Merge

1. **Linting passes**: `yarn lint`
2. **Type checking passes**: `yarn tsc`
3. **Tests pass**: `yarn test`
4. **Build succeeds**: `yarn build`

---

## Common Pitfalls to Avoid

### 1. Direct State Mutation

```typescript
// Wrong
user.name = "New Name";

// Correct
@action
updateName(name: string) {
  this.name = name;
}
```

### 2. Missing Error Handling

```typescript
// Wrong
async function fetchData() {
  const data = await api.get("/data");
  return data;
}

// Correct
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

### 3. Unnecessary React Import

```tsx
// Wrong - React import not needed
import React from "react";

function Component() {
  return <div>Hello</div>;
}

// Correct
function Component() {
  return <div>Hello</div>;
}
```

### 4. Deep Relative Imports

```typescript
// Wrong
import { helper } from "../../../shared/utils/helper";

// Correct
import { helper } from "@shared/utils/helper";
```

### 5. Missing JSDoc on Public Functions

```typescript
// Wrong
export function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// Correct
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
// Wrong
if (value == null) { ... }

// Correct
if (value === null || value === undefined) { ... }
```

### 7. Missing Curly Braces

```typescript
// Wrong
if (condition) return value;

// Correct
if (condition) {
  return value;
}
```

### 8. Non-Self-Closing Tags

```tsx
// Wrong
<div className="spacer"></div>

// Correct
<div className="spacer" />
```

### 9. Forgetting to Use Presenters

```typescript
// Wrong - returning raw model data
ctx.body = { data: document };

// Correct - using presenter
ctx.body = { data: presentDocument(document) };
```

### 10. Skipping Authorization Checks

```typescript
// Wrong - no authorization
router.post("documents.delete", auth(), async (ctx) => {
  const document = await Document.findByPk(id);
  await document.destroy();
});

// Correct - with authorization
router.post("documents.delete", auth(), async (ctx) => {
  const { user } = ctx.state.auth;
  const document = await Document.findByPk(id);
  authorize(user, "delete", document);
  await document.destroy();
});
```

---

## Development Commands

### Essential Commands

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

# Build for production
yarn build
```

### Database Commands

```bash
# Run migrations
yarn db:migrate

# Rollback last migration
yarn db:rollback

# Create new migration
yarn db:create-migration <name>
```

### Testing Commands

```bash
# Run specific test
yarn test path/to/file.test.ts

# Run frontend tests
yarn test:app

# Run backend tests (requires PostgreSQL)
yarn test:server

# Run shared tests
yarn test:shared

# Run with coverage
yarn test --coverage
```

### Docker Commands

```bash
# Build Docker image
docker build -t outline .

# Run with Docker Compose (local development)
docker-compose up -d
```

---

## CI/CD Pipeline

### GitHub Actions Checks

The CI pipeline (`.github/workflows/ci.yml`) runs the following checks on every PR:

| Job | Description | Command |
|-----|-------------|---------|
| `lint` | Code quality checks | `yarn lint --quiet` |
| `types` | TypeScript type checking | `yarn tsc` |
| `test` | Frontend tests (app, shared) | `yarn test:app`, `yarn test:shared` |
| `test-server` | Backend tests (4 shards) | `yarn test:server` |
| `bundle-size` | Track bundle size changes | RelativeCI |

### Preview Environments

Preview environments are automatically created for pull requests via Render. The `render.yaml` configuration enables:

- Automatic preview deployment for every PR
- Same configuration as production (Docker-based)
- Synced environment variables from production

---

## Additional Resources

- **Architecture**: See `docs/ARCHITECTURE.md` for detailed system architecture
- **API Documentation**: https://getoutline.com/developers
- **Hosting Guide**: https://docs.getoutline.com/s/hosting
- **Contributing Guide**: See `CONTRIBUTING.md` for contribution guidelines

---

## Quick Reference Card

### Before Making Changes
1. Read related code thoroughly
2. Check existing patterns in similar files
3. Understand the data flow

### While Coding
1. Use path aliases (`@server`, `@shared`, `~`)
2. Add JSDoc to public functions
3. Use strict equality (`===`)
4. Handle null/undefined cases
5. Use presenters for API responses
6. Check authorization with policies

### Before Committing
1. Run `yarn lint`
2. Run `yarn tsc`
3. Run relevant tests
4. Write descriptive commit message

### Before Creating PR
1. All checks pass locally
2. Tests added for new functionality
3. Documentation updated if needed
4. PR description is complete
