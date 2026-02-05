# AI Agent Workflows for Outline

> **For comprehensive coding standards, architecture details, and style guidelines, see [`CLAUDE.md`](./CLAUDE.md).** This document focuses on operational workflows and checklists for AI coding agents.

## Overview

Outline is a fast, collaborative knowledge base for teams — a TypeScript monorepo with a React frontend, Koa backend, Prosemirror editor, and real-time collaboration via Y.js. See `CLAUDE.md` for the full architecture breakdown.

## Repository Structure (Quick Reference)

| Directory    | Purpose                                      |
| ------------ | -------------------------------------------- |
| `app/`       | React frontend (MobX, styled-components)     |
| `server/`    | Koa API server (Sequelize, PostgreSQL, Redis) |
| `shared/`    | Code shared between frontend and backend     |
| `plugins/`   | Plugin system for extending functionality    |

## Agent Workflow Checklist

### Before Writing Code

1. Read `CLAUDE.md` for coding standards and patterns
2. Understand the existing patterns in the area you are modifying
3. Use path aliases (`@server/*`, `@shared/*`, `~/*`) — never deep relative imports

### Before Committing

1. **Lint**: Always run `yarn lint` and fix any issues
2. **Type check**: Run `yarn tsc` to catch type errors
3. **Test**: Run tests on the areas you modified:
   - Specific file: `yarn test path/to/file.test.ts`
   - Frontend suite: `yarn test:app`
   - Backend suite: `yarn test:server`
   - Shared suite: `yarn test:shared`
4. **Format**: Run `yarn format` to ensure consistent formatting

### After Committing

1. Verify the commit includes only the intended files
2. Ensure no secrets, `.env` files, or credentials are committed

## Testing Requirements

| Environment | Command            | Runner       | Notes                               |
| ----------- | ------------------ | ------------ | ----------------------------------- |
| Frontend    | `yarn test:app`    | Jest + jsdom | React Testing Library for components |
| Backend     | `yarn test:server` | Jest + Node  | Uses real PostgreSQL for integration |
| Shared      | `yarn test:shared` | Jest (both)  | Runs in both jsdom and Node          |
| Specific    | `yarn test --testPathPattern=<path>` | Jest | Preferred for targeted testing |

- Tests are **colocated** with source files (`*.test.ts` / `*.test.tsx`)
- **Do not** create new test directories
- Mock external dependencies in `__mocks__/` folders
- Every bug fix should include a regression test

## Common Patterns

### API Routes

- Routes live in `server/routes/api/` and follow RESTful conventions
- **Authorization**: Use `@server/policies` (cancan) — always check permissions before operations
- **Validation**: Use validation middleware for all request data
- **Response formatting**: Use `@server/presenters` — never return raw model data
- **Business logic**: Use `@server/commands` for complex operations that span multiple models

### Events System

- Side effects (notifications, webhooks, audit logs) use the events system
- Emit events from commands rather than directly from route handlers
- Event processors live in `server/queues/`

### Frontend State

- Use MobX `@action` decorators for all state mutations
- Use `@computed` for derived state
- Stores in `app/stores/` manage collections of models
- Models in `app/models/` are MobX observables

### Plugin Architecture

- New features should follow the plugin pattern in `plugins/`
- Plugins register routes, models, and UI components
- Look at existing plugins for reference implementations

### Internationalization

- All user-facing strings must use the i18n system (`shared/i18n/`)
- Update translation files when adding new strings
- Never hardcode user-visible text

## Security Considerations

- **SQL injection**: Always use Sequelize parameterized queries — never raw string concatenation
- **Authorization**: Every API endpoint must check policies before performing operations
- **Input validation**: Validate and sanitize all user input via validation middleware
- **XSS prevention**: Use React's built-in escaping; avoid `dangerouslySetInnerHTML`
- **Secrets**: Never commit `.env` files, API keys, or credentials

## Code Review Checklist (Agent Self-Review)

Before submitting changes, verify:

- [ ] No unused imports or variables
- [ ] Proper error handling on all async operations
- [ ] i18n compliance for user-facing strings
- [ ] Accessibility: ARIA roles, semantic HTML, keyboard navigation
- [ ] Authorization checks in API routes
- [ ] Tests added or updated for the changes
- [ ] `yarn lint` passes cleanly
- [ ] `yarn tsc` passes cleanly
- [ ] No `console.log` statements in production code
- [ ] JSDoc on all public/exported functions

## Pull Request Guidelines

- Use conventional commit format for PR titles (e.g., `feat:`, `fix:`, `refactor:`)
- Keep changes small and focused — one concern per PR
- Include a test plan describing how changes were verified
- Update translations if adding user-facing strings
- Reference related issues in the PR description
- See `CLAUDE.md` → "Pull Request Guidelines" for full PR description format
