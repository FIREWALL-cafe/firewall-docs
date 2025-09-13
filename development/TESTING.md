# Testing Guide

> Last Updated: 2025-01-12
> Framework: Jest + React Testing Library
> Test Files: [`firewall-client-server/client/src/__tests__/`](https://github.com/FIREWALL-cafe/firewall-client-server/tree/main/client/src/__tests__)

## Overview

Firewall Cafe uses Jest and React Testing Library for frontend testing. The current test coverage is minimal, providing an opportunity for significant contribution.

## Test Infrastructure

### Configuration
- **Jest Config**: [`client/package.json`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/package.json) - See "jest" section
- **Test Scripts**:
  - `yarn test` - Run tests in watch mode
  - `yarn test:ci` - Single run for CI/CD
  - `yarn test:minimal` - Run minimal test suite

### Current Test Coverage
- **Minimal Test**: [`client/src/__tests__/TermsAndConditions.minimal.test.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/__tests__/TermsAndConditions.minimal.test.js)
- **Coverage**: Very limited - primarily a placeholder test

## Running Tests

### Development Testing
```bash
cd client
yarn test
```

### Continuous Integration
```bash
cd client
yarn test:ci
```

### Pre-deployment Checks
```bash
cd client
yarn format:check
yarn lint
yarn test:ci
yarn build
```

## Test Patterns

### Component Testing
- **Location**: `client/src/__tests__/components/`
- **Pattern**: One test file per component
- **Naming**: `ComponentName.test.js`

### Integration Testing
- **Location**: `client/src/__tests__/integration/`
- **Focus**: User flows and feature interactions

### API Testing
- **Location**: `server/api/__tests__/`
- **Framework**: Jest with supertest for endpoint testing

## Key Areas Needing Tests

### Frontend Components
1. **Search Components**: `Search.jsx`, `SearchInput.jsx`, `SearchResults.jsx`
2. **Filter System**: `FilterControls.jsx` - Complex state and validation logic
3. **Dashboard**: Analytics components and data visualization
4. **Archive**: Pagination and search result display

### API Endpoints
1. **Search Operations**: `/images`, `/searches` endpoints
2. **Analytics**: Geographic and vote analytics endpoints
3. **Voting System**: Vote submission and retrieval

### Integration Points
1. **Search Flow**: End-to-end search comparison
2. **Filter Application**: Complex filter combinations
3. **Vote Submission**: User interaction flow

## Testing Best Practices

### Component Testing
- Test user interactions, not implementation details
- Use React Testing Library queries (getByRole, getByText)
- Mock external dependencies
- Test accessibility features

### API Testing
- Test both success and error cases
- Validate response schemas
- Test pagination and filtering
- Mock external services (Serper, Baidu, etc.)

### Performance Testing
- Monitor bundle size changes
- Test component render performance
- Validate API response times

## Development Workflow

### Writing New Tests
1. Create test file adjacent to component
2. Follow existing patterns in codebase
3. Run tests locally before committing
4. Ensure CI passes before merging

### Test Data
- **Fixtures**: Store in `__tests__/fixtures/`
- **Mocks**: Store in `__tests__/mocks/`
- **Utilities**: Store in `__tests__/utils/`

## Quality Assurance

### Linting
- **ESLint Config**: [`client/.eslintrc.json`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/.eslintrc.json)
- **Run**: `yarn lint` or `yarn lint:fix`

### Code Formatting
- **Prettier Config**: [`client/.prettierrc`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/.prettierrc)
- **Run**: `yarn format` or `yarn format:check`

## Future Improvements

### Coverage Goals
- Achieve 80% code coverage for critical paths
- 100% coverage for utility functions
- Integration tests for all major user flows

### Testing Infrastructure
- Add E2E testing with Playwright or Cypress
- Implement visual regression testing
- Add performance benchmarking

### Documentation
- Document test data requirements
- Create testing guidelines for contributors
- Add test examples for common patterns

---

*For development workflow, see [Development Workflow](./WORKFLOW.md)*
*For contribution guidelines, see [Contributing](./CONTRIBUTING.md)*