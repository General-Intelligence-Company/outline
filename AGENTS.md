# AI Coding Agent Guidelines

This document provides guidelines for AI coding agents working on the Outline codebase. It complements CLAUDE.md with additional context for automated code generation and review.

## Project Overview

Outline is a fast, collaborative knowledge base for teams, built with:
- **Frontend**: React + TypeScript + MobX + Styled Components
- **Backend**: Koa + Sequelize ORM + PostgreSQL + Redis
- **Real-time**: WebSockets + Y.js for collaborative editing

## Codebase Navigation

### Key Directories

```
app/                    # React frontend application
├── actions/           # Reusable actions (navigate, create entities)
├── components/        # Shared React components
├── editor/            # Editor-specific components
├── hooks/             # Custom React hooks
├── menus/             # Context menus
├── models/            # MobX observable models
├── routes/            # Route definitions (lazy-loaded)
├── scenes/            # Full-page view components
├── stores/            # MobX stores and fetch logic
├── types/             # TypeScript type definitions
└── utils/             # Frontend utility methods

server/                 # Koa backend server
├── routes/            # API and auth routes
│   ├── api/          # RESTful API endpoints
│   └── auth/         # Authentication routes
├── commands/          # Multi-model business logic
├── emails/            # Email templates
├── middlewares/       # Koa middlewares
├── migrations/        # Database migrations
├── models/            # Sequelize models
├── policies/          # Authorization (cancan-style)
├── presenters/        # JSON response formatters
├── queues/            # Async job processing
│   ├── processors/   # Event bus processors
│   └── tasks/        # Standalone async tasks
└── utils/             # Backend utilities

shared/                 # Code shared between frontend/backend
├── components/        # Shared React components
├── editor/            # Prosemirror editor core
├── i18n/              # Internationalization
├── styles/            # Global styles and colors
└── utils/             # Shared utilities

plugins/                # Plugin system extensions
```

### Finding Code

- **Components**: Start in `app/components/` for reusable UI
- **Pages/Views**: Check `app/scenes/` for full-page components
- **API Endpoints**: Look in `server/routes/api/`
- **Database Models**: Found in `server/models/`
- **State Management**: MobX stores in `app/stores/`
- **Editor Logic**: Prosemirror code in `shared/editor/`

## Pre-Commit Requirements

Before committing any changes, ensure all checks pass:

```bash
# Required checks (run all before committing)
yarn lint          # Oxlint - must pass with no errors
yarn format        # Prettier - auto-formats code
yarn tsc           # TypeScript - must compile without errors
yarn test          # Jest - all tests must pass

# Run specific test file (preferred for development)
yarn test path/to/file.test.ts
```

## Code Standards

### TypeScript Rules

- **Strict mode** is enforced
- **Never use `any`** - use proper types or generics
- **Avoid `unknown`** unless absolutely necessary
- **No type assertions** (`as`, `!`) - prefer type guards
- **Always use curly braces** for if/else statements
- **Prefer `interface`** over `type` for object shapes

### React Patterns

```typescript
// ✅ Correct: Functional component with typed props
interface ButtonProps {
  onClick: () => void;
  disabled?: boolean;
  children: React.ReactNode;
}

export function Button({ onClick, disabled, children }: ButtonProps) {
  const handleClick = () => {
    onClick();
  };

  return (
    <StyledButton onClick={handleClick} disabled={disabled}>
      {children}
    </StyledButton>
  );
}

// ❌ Incorrect: Class component, any types, missing interface
export class Button extends React.Component<any> { ... }
```

### Event Handler Naming

- Prefix with `handle`: `handleClick`, `handleSubmit`, `handleChange`
- Match the event: `onClick` → `handleClick`

### Class Member Order

1. Public static variables
2. Public static methods
3. Public variables
4. Public methods
5. Protected variables & methods
6. Private variables & methods

### Exports

- Exported members at the **top** of the file
- Prefer **named exports** for components and classes
- **JSDoc required** for all public/exported functions

## Testing Requirements

### Test File Location

- Tests are **co-located** with source files
- Name pattern: `*.test.ts` or `*.test.tsx`
- Do **not** create new test directories

### Running Tests

```bash
# Preferred: Run specific test file
yarn test server/models/User.test.ts

# Test suites (use sparingly)
yarn test:app      # Frontend tests
yarn test:server   # Backend tests
yarn test:shared   # Shared code tests
```

### Test Coverage

- Minimum **60%** coverage required for branches, functions, lines, statements
- Focus on **critical paths** and business logic
- Mock external dependencies in `__mocks__/` folder

## Code Review Checklist

Before submitting code, verify:

### Functionality
- [ ] Code compiles without errors (`yarn tsc`)
- [ ] All tests pass (`yarn test`)
- [ ] No linting errors (`yarn lint`)
- [ ] Code is properly formatted (`yarn format`)

### Code Quality
- [ ] No `any` types used
- [ ] Proper error handling implemented
- [ ] JSDoc comments on public functions
- [ ] Early returns used for readability
- [ ] No console.log statements (use proper logging)

### Security
- [ ] User input is sanitized
- [ ] Authorization checks in place (policies)
- [ ] No sensitive data exposed in errors
- [ ] Rate limiting on sensitive endpoints

### Performance
- [ ] Database queries use appropriate indexes
- [ ] React components use memoization where needed
- [ ] Large lists implement pagination
- [ ] No N+1 query problems

## Common Pitfalls

### Avoid These Mistakes

1. **Creating new markdown files** - The project specifically prohibits this
2. **Using `any` types** - Always define proper types
3. **Forgetting JSDoc** - All exports need documentation
4. **Smart quote replacement** - Don't replace `""` or `''` with `""`
5. **Manual translation strings** - They're extracted automatically
6. **Using `#` for private properties** - Use TypeScript `private` keyword
7. **Skipping pre-commit hooks** - Always run `yarn lint`, `yarn format`, `yarn tsc`

### Database Operations

```typescript
// ✅ Use transactions for multi-table operations
await sequelize.transaction(async (transaction) => {
  await User.create({ name: "test" }, { transaction });
  await Team.create({ userId: user.id }, { transaction });
});

// ❌ Don't perform multi-table operations without transactions
await User.create({ name: "test" });
await Team.create({ userId: user.id }); // May fail, leaving orphaned User
```

### MobX State

```typescript
// ✅ Keep business logic in stores
class DocumentStore {
  @action
  async fetchDocument(id: string) {
    const doc = await client.get(`/api/documents/${id}`);
    this.add(doc);
  }
}

// ❌ Don't put business logic in components
function DocumentView() {
  useEffect(() => {
    fetch(`/api/documents/${id}`).then(...); // Move to store
  }, []);
}
```

## Integration Points

### Adding New API Endpoints

1. Create route in `server/routes/api/`
2. Add validation schema
3. Create/update policy in `server/policies/`
4. Add presenter in `server/presenters/`
5. Write tests in co-located `.test.ts` file

### Adding New React Components

1. Create component in appropriate directory
2. Use styled-components for styling
3. Define TypeScript interface for props
4. Add JSDoc documentation
5. Implement accessibility (ARIA roles, semantic HTML)
6. Write tests in co-located `.test.tsx` file

### Database Changes

1. Generate migration: `yarn sequelize migration:create --name=description`
2. Update model in `server/models/`
3. Run migration: `yarn db:migrate`
4. Add appropriate indexes
5. Update related tests

## Error Handling

```typescript
// Use custom error classes from server/errors.ts
import { NotFoundError, ValidationError } from "@server/errors";

// ✅ Proper error handling
async function getDocument(id: string) {
  const document = await Document.findByPk(id);
  if (!document) {
    throw NotFoundError("Document not found");
  }
  return document;
}

// ✅ Graceful error responses
router.get("/documents/:id", async (ctx) => {
  try {
    ctx.body = await getDocument(ctx.params.id);
  } catch (error) {
    if (error instanceof NotFoundError) {
      ctx.status = 404;
      ctx.body = { error: error.message };
    } else {
      throw error; // Let middleware handle unexpected errors
    }
  }
});
```

## Quick Reference

| Task | Command |
|------|---------|
| Install dependencies | `yarn install` |
| Run linter | `yarn lint` |
| Format code | `yarn format` |
| Type check | `yarn tsc` |
| Run all tests | `yarn test` |
| Run specific test | `yarn test path/to/file.test.ts` |
| Create migration | `yarn sequelize migration:create --name=name` |
| Run migrations | `yarn db:migrate` |

## Additional Resources

- Architecture details: `/docs/ARCHITECTURE.md`
- Coding standards: `/CLAUDE.md`
- API documentation: https://getoutline.com/developers
