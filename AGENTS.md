# AI Agent Guidelines for Outline

This document provides guidelines for AI coding agents working on the Outline codebase.

## Quick Reference

- **Language**: TypeScript (strict null checks enabled)
- **Package Manager**: Yarn 4.11.0 (use `yarn` commands, not `npm`)
- **Linter**: oxlint (`yarn lint`)
- **Formatter**: Prettier (`yarn prettier`)
- **Test Framework**: Jest with sharding
- **Backend**: Node.js with Koa framework
- **Frontend**: React with MobX state management
- **Editor**: Prosemirror with Y.js real-time collaboration

## Before Making Changes

1. **Read CLAUDE.md first** - Contains detailed architecture and patterns
2. **Run type check**: `yarn tsc` before committing
3. **Run lint**: `yarn lint --quiet` to check for issues
4. **Run tests**: `yarn test` for the relevant area

## Code Style Requirements

### TypeScript
- Use strict null checks - always handle `null` and `undefined` cases
- Prefer `interface` over `type` for object shapes
- Use explicit return types for exported functions
- No `any` types - use `unknown` if type is truly unknown

### React Components
- Use functional components with hooks
- Use MobX `observer` for reactive components
- Follow existing component patterns in `app/components/`
- Use `styled-components` for styling (following existing patterns)

### Backend (Server)
- Follow existing patterns in `server/` directory
- Use Koa middleware patterns
- Transactions must be properly handled with Sequelize
- Events should be dispatched for significant actions

### Database
- Migrations go in `server/migrations/`
- Models extend BaseModel in `server/models/`
- Always include proper indexes for new queries
- Use transactions for multi-step operations

## Testing Requirements

### Before Submitting Code
- [ ] All existing tests pass (`yarn test`)
- [ ] New functionality has tests
- [ ] Type check passes (`yarn tsc`)
- [ ] Lint passes (`yarn lint --quiet`)

### Test Locations
- Server tests: `server/**/*.test.ts`
- App tests: `app/**/*.test.ts`
- Shared tests: `shared/**/*.test.ts`
- Plugin tests: `plugins/**/*.test.ts`

### Running Specific Tests
```bash
# Run all tests
yarn test

# Run server tests only
yarn test:server

# Run specific test file
yarn jest path/to/test.test.ts
```

## Common Patterns

### Adding a New API Endpoint
1. Create route in `server/routes/api/`
2. Add authentication/authorization middleware
3. Use existing presenter patterns for responses
4. Add appropriate tests
5. Dispatch events for auditing

### Adding a New React Component
1. Create in `app/components/` following existing patterns
2. Use `observer` wrapper if using MobX stores
3. Use `styled-components` for styling
4. Add prop types/interfaces

### Adding a New Plugin
1. Follow structure in `plugins/` directory
2. Include `plugin.json` manifest
3. Register in appropriate hooks

## Code Review Checklist

- [ ] No TypeScript errors (`yarn tsc`)
- [ ] No lint errors (`yarn lint --quiet`)
- [ ] Tests added for new functionality
- [ ] Existing tests still pass
- [ ] No hardcoded strings (use i18n)
- [ ] Proper error handling
- [ ] No console.log statements in production code
- [ ] Database migrations are reversible
- [ ] API changes are backward compatible

## Performance Considerations

- Avoid N+1 queries - use eager loading
- Use pagination for list endpoints
- Consider caching for expensive operations
- Profile large changes with the built-in tools

## Security Guidelines

- Never trust user input - always validate
- Use parameterized queries (Sequelize handles this)
- Check authorization before every sensitive operation
- Don't expose internal IDs unnecessarily
- Sanitize HTML content

## Debugging Tips

- Use `DEBUG=*` environment variable for verbose logging
- Check `server/logs/` for application logs
- Use browser DevTools for frontend debugging
- React DevTools and MobX DevTools are helpful

## Getting Help

- Read CLAUDE.md for detailed architecture
- Check existing code for patterns
- Look at recent PRs for examples
- Consult the external docs at https://getoutline.com/developers
