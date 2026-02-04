# AI Agent Guidelines for Outline

This document provides guidance for AI agents working on the Outline codebase. For detailed architecture, code patterns, and comprehensive examples, see [`CLAUDE.md`](./CLAUDE.md).

## Overview

Outline is a collaborative knowledge base built with TypeScript. AI agents working on this codebase should:

1. **Read CLAUDE.md first** - Contains detailed architecture, patterns, and code examples
2. **Run validation before committing** - Lint, type-check, and test your changes
3. **Follow existing patterns** - Match the style of surrounding code
4. **Keep changes focused** - One logical change per PR

## CI/CD Pipeline

### Automated Checks

All PRs must pass these CI checks before merging:

| Check | Command | Description |
|-------|---------|-------------|
| **Lint** | `yarn lint` | Oxlint static analysis |
| **Types** | `yarn tsc` | TypeScript type checking |
| **Tests (app)** | `yarn test:app` | Frontend tests (jsdom) |
| **Tests (server)** | `yarn test:server` | Backend tests (node) |
| **Tests (shared)** | `yarn test:shared` | Shared code tests |

### Path-Based Test Execution

CI runs tests selectively based on changed files:
- **app/** or **shared/** changes: Runs `test:app` and `test:shared`
- **server/** or **shared/** changes: Runs `test:server`
- Server tests are sharded across 4 parallel jobs for performance

### Security Scanning

- **CodeQL**: Automated security analysis on all PRs and weekly scheduled scans
- **Dependabot**: Weekly dependency updates with grouped PRs for related packages (Babel, Sentry, AWS SDK, Radix UI, FontAwesome)

### Additional Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| Image Compression | PR with images | Automatically compresses images (10%+ size reduction) |
| Stale PR Closure | Daily | Marks PRs stale after 60 days, closes after 5 more days |
| CLA Enforcement | Daily | Closes PRs without signed CLA after 14 days |
| Docker Build | Version tags (`v*`) | Multi-arch Docker image builds (amd64/arm64) |

## Pre-Commit Validation

Before committing, always run:

```bash
# All three should pass
yarn lint        # Linting (Oxlint)
yarn tsc         # Type checking
yarn test        # Run relevant tests
```

## Code Review Checklist

When reviewing code changes, verify:

### TypeScript
- [ ] No `any` type usage
- [ ] `interface` preferred over `type` for objects
- [ ] Minimal type assertions (`as`, `!`)
- [ ] Proper null/undefined handling
- [ ] `import type` for type-only imports

### React
- [ ] Functional components only
- [ ] Event handlers prefixed with `handle`
- [ ] styled-components for styling
- [ ] Self-closing tags for empty elements
- [ ] No unnecessary `React` import

### Code Quality
- [ ] Early returns for readability
- [ ] Curly braces on all `if` statements
- [ ] JSDoc on public/exported functions
- [ ] No `console.log` in production code
- [ ] Strict equality (`===`) only
- [ ] Path aliases (`@server/`, `@shared/`, `~/`)

### API Routes
- [ ] Request validation middleware used
- [ ] Response formatting via presenters
- [ ] Authorization via policies
- [ ] Proper error handling

### Database
- [ ] Backward-compatible migrations
- [ ] Rollback migration provided
- [ ] Indexes on queried columns

### Tests
- [ ] Tests colocated with source (`.test.ts`)
- [ ] Coverage thresholds maintained (50% statements, 40% branches)
- [ ] Critical paths covered

## Testing Requirements

### Coverage Thresholds

| Metric | Minimum |
|--------|---------|
| Statements | 50% |
| Branches | 40% |
| Functions | 50% |
| Lines | 50% |

### Test File Location

Tests must be colocated with source files:

```
server/models/User.ts
server/models/User.test.ts    # Correct
server/test/User.test.ts      # Wrong - don't create test directories
```

### What to Test

| Change Type | Required Tests |
|-------------|----------------|
| New API endpoint | Route handler unit tests, integration tests |
| New React component | Rendering tests, interaction tests |
| Business logic | Unit tests for commands/utilities |
| Database model | Validation tests, association tests |
| Bug fix | Regression test proving the fix |

## Common Tasks

### Adding a New API Endpoint

1. Create route handler in `server/routes/api/`
2. Add request validation using validation middleware
3. Create/update presenter in `server/presenters/`
4. Add authorization policy in `server/policies/`
5. Write tests colocated with the route handler
6. Run `yarn lint && yarn tsc && yarn test:server`

### Adding a New React Component

1. Create component in appropriate `app/` directory
2. Use styled-components for styling
3. Use functional component with hooks
4. Add tests in same directory with `.test.tsx` extension
5. Run `yarn lint && yarn tsc && yarn test:app`

### Database Migrations

```bash
# Create a new migration
yarn db:create-migration --name add_column_to_users

# Run migrations
yarn db:migrate

# Rollback if needed
yarn db:rollback
```

Always:
- Make migrations backward compatible
- Provide a rollback migration
- Add indexes for frequently queried columns

## Security Considerations

### Authentication & Authorization

- Always use cancan policies for authorization checks
- Never bypass policy checks even for "admin" operations
- Validate all user input through validation middleware

### Data Handling

- Never log sensitive data (passwords, tokens, PII)
- Use presenters to control what data is exposed in API responses
- Sanitize user input before database operations

### Dependencies

- Review Dependabot PRs for security implications
- Check for security advisories in updated packages
- Prefer packages with active maintenance

### Secrets

- Never commit secrets, API keys, or credentials
- Use environment variables for configuration
- Check `.env.example` for required variables

## Do's and Don'ts

### Do

- Run `yarn lint && yarn tsc && yarn test` before committing
- Use path aliases (`@server/`, `@shared/`, `~/`)
- Write JSDoc for public functions
- Handle errors with try/catch and proper logging
- Use MobX `@action` decorators for state mutations
- Use `@computed` for derived state
- Follow conventional commit format for PR titles
- Keep PRs focused on a single logical change

### Don't

- Use `any` type
- Use relative imports for cross-directory references
- Skip authorization policy checks
- Commit `console.log` statements
- Mutate MobX observables directly
- Create separate test directories
- Use loose equality (`==`)
- Skip curly braces on `if` statements
- Import React when only using JSX (JSX transform is enabled)

## Pull Request Guidelines

### PR Title Format

Use conventional commits:
- `feat: Add user profile editing`
- `fix: Resolve document loading issue`
- `refactor: Simplify authentication flow`
- `docs: Update API documentation`
- `test: Add missing model tests`
- `chore: Update dependencies`

### PR Description

Include:
1. **Summary**: What changed and why
2. **Testing**: How changes were verified
3. **Screenshots**: For UI changes
4. **Breaking Changes**: Note any breaking changes

### CLA Requirement

All contributors must sign the CLA. PRs without signed CLA will be automatically closed after 14 days.

## Additional Resources

- **Detailed patterns and examples**: [`CLAUDE.md`](./CLAUDE.md)
- **API documentation**: https://getoutline.com/developers
- **System architecture**: `docs/ARCHITECTURE.md`
