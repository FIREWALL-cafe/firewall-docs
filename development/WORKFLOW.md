# Development Workflow

This guide covers the development workflow, best practices, and common tasks for contributing to Firewall Cafe.

## Table of Contents
- [Git Workflow](#git-workflow)
- [Code Standards](#code-standards)
- [Development Scripts](#development-scripts)
- [Testing Strategy](#testing-strategy)
- [Code Review Process](#code-review-process)
- [Release Process](#release-process)

## Git Workflow

### Branch Strategy

We follow a **feature branch workflow** with the following conventions:

```
main                    ← Production-ready code
├── feature/search-ui   ← New feature development
├── bugfix/vote-count   ← Bug fixes
├── hotfix/security     ← Critical production fixes
└── docs/update-readme  ← Documentation updates
```

### Branch Naming Convention

- **Features**: `feature/short-description`
- **Bug Fixes**: `bugfix/short-description`
- **Hotfixes**: `hotfix/short-description`
- **Documentation**: `docs/short-description`
- **Refactoring**: `refactor/short-description`

### Commit Message Format

Follow conventional commits for clear history:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples:**
```bash
feat(search): add real-time Google/Baidu comparison
fix(api): resolve PostgreSQL connection pool timeout
docs(setup): update development environment guide
refactor(components): extract SearchForm into reusable component
```

### Development Process

#### 1. Start New Work
```bash
# Ensure you're on main and up to date
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/your-feature-name

# Verify Node version
nvm use  # Uses version from .nvmrc (Node 20.x)
```

#### 2. Development Loop
```bash
# Install dependencies (if needed)
cd client
yarn install

# Start development servers
yarn dev  # Starts both client and server with hot reload

# Make your changes...
# Run linting and formatting frequently
yarn lint:fix
yarn format
```

#### 3. Testing Your Changes
```bash
# Run tests
yarn test:minimal  # Quick test suite
yarn test:ci      # Full test suite

# Manual testing checklist:
# □ Search functionality works
# □ Analytics dashboard loads
# □ Heatmaps render correctly
# □ Voting system responds
# □ Responsive design works
```

#### 4. Commit and Push
```bash
# Stage your changes
git add .

# Commit with descriptive message
git commit -m "feat(heatmap): add US state-level visualization"

# Push feature branch
git push origin feature/your-feature-name
```

#### 5. Create Pull Request
1. Go to GitHub repository
2. Create pull request from your feature branch to `main`
3. Fill out PR template with:
   - **Description**: What changes were made
   - **Testing**: How to test the changes
   - **Screenshots**: For UI changes
   - **Checklist**: Confirm all requirements met

### SSH Agent Setup (for Remote Work)

When working on remote servers or accessing private repositories:

```bash
# Start SSH agent
eval "$(ssh-agent)"

# Add your key (macOS)
ssh-add -K ~/.ssh/id_rsa

# SSH with agent forwarding
ssh -A user@server

# Test GitHub access from remote
ssh -T git@github.com
```

## Code Standards

### JavaScript/React Standards

#### Code Style
- Use **ES6+ syntax** (arrow functions, destructuring, modules)
- Use **functional components** with hooks over class components
- Keep components **small and focused** (< 200 lines)
- Use **meaningful variable names** (avoid abbreviations)

#### React Patterns
```javascript
// ✅ Good: Functional component with proper hooks
import React, { useState, useEffect } from 'react';

function SearchComparison({ query, onResultsChange }) {
  const [results, setResults] = useState(null);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    if (query) {
      fetchSearchResults(query);
    }
  }, [query]);
  
  const fetchSearchResults = async (searchQuery) => {
    setLoading(true);
    try {
      const data = await api.search(searchQuery);
      setResults(data);
      onResultsChange(data);
    } catch (error) {
      console.error('Search failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) return <LoadingSpinner />;
  
  return (
    <div className="search-comparison">
      {/* Component content */}
    </div>
  );
}

export default SearchComparison;
```

#### State Management
- Use **React Context** for global state (see `ApiContext`)
- Use **useState** for local component state
- Use **useEffect** for side effects and API calls
- **Avoid** prop drilling beyond 2-3 levels

### CSS/Styling Standards

#### Tailwind CSS Best Practices
```javascript
// ✅ Good: Organized, semantic classes
<div className="
  flex flex-col items-center justify-center
  p-6 mx-auto max-w-4xl
  bg-white rounded-lg shadow-md
  dark:bg-gray-800
">

// ❌ Avoid: Long, unorganized class strings
<div className="flex p-6 bg-white items-center mx-auto max-w-4xl justify-center flex-col rounded-lg shadow-md dark:bg-gray-800">
```

#### Component-Specific Styles
- Use Tailwind classes for most styling
- Create custom CSS only when Tailwind is insufficient
- Use CSS modules or styled-components for complex animations

### File Organization

#### Directory Structure
```
src/
├── components/           # Reusable UI components
│   ├── common/          # Shared components (Button, Modal, etc.)
│   ├── search/          # Search-related components
│   ├── analytics/       # Analytics components
│   └── maps/           # Geographic/heatmap components
├── pages/              # Route-level components
├── hooks/              # Custom React hooks
├── context/            # React Context providers
├── utils/              # Utility functions
├── constants/          # Configuration and mappings
└── assets/            # Images, icons, static files
```

#### Component File Structure
```javascript
// SearchForm.jsx
import React, { useState } from 'react';
import PropTypes from 'prop-types';

// Component implementation
const SearchForm = ({ onSearch, placeholder = "Enter search term..." }) => {
  // Component code
};

// PropTypes for documentation and validation
SearchForm.propTypes = {
  onSearch: PropTypes.func.isRequired,
  placeholder: PropTypes.string,
};

export default SearchForm;
```

## Development Scripts

### Available Scripts

#### Development
```bash
yarn dev                 # Start both client and server
yarn start:client        # Start React dev server only
yarn start:server        # Start Express server only
```

#### Testing
```bash
yarn test               # Run tests in watch mode
yarn test:ci            # Run all tests once (CI mode)
yarn test:minimal       # Quick test suite for rapid feedback
```

#### Code Quality
```bash
yarn lint               # Check for linting errors
yarn lint:fix           # Fix auto-fixable linting errors
yarn format             # Format code with Prettier
yarn format:check       # Check if code is properly formatted
```

#### Build & Deploy
```bash
yarn build              # Create production build
yarn start              # Start production server
yarn deploy             # Deploy to GitHub Pages (if configured)
```

### Environment Configuration

#### Development Environment
```bash
# client/.env.development
REACT_APP_API_URL=http://localhost:8080
REACT_APP_DEBUG_MODE=true
REACT_APP_ENABLE_HOT_RELOAD=true
```

#### Production Environment  
```bash
# client/.env.production
REACT_APP_API_URL=https://api.firewallcafe.com
REACT_APP_DEBUG_MODE=false
REACT_APP_ENABLE_ANALYTICS=true
```

## Testing Strategy

### Test Pyramid

1. **Unit Tests** (70%)
   - Individual functions and components
   - React component rendering
   - Utility function logic

2. **Integration Tests** (20%)
   - API integration
   - Component interaction
   - Context providers

3. **End-to-End Tests** (10%)
   - Complete user workflows
   - Cross-browser compatibility
   - Performance testing

### Testing Tools

- **Jest**: Test runner and assertion library
- **React Testing Library**: Component testing utilities
- **MSW**: API mocking for integration tests
- **Playwright**: E2E testing (when needed)

### Example Tests

#### Component Test
```javascript
// SearchForm.test.jsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import SearchForm from './SearchForm';

describe('SearchForm', () => {
  const mockOnSearch = jest.fn();

  beforeEach(() => {
    mockOnSearch.mockClear();
  });

  test('calls onSearch when form is submitted', async () => {
    render(<SearchForm onSearch={mockOnSearch} />);
    
    const input = screen.getByRole('textbox');
    const submitButton = screen.getByRole('button', { name: /search/i });
    
    fireEvent.change(input, { target: { value: 'test query' } });
    fireEvent.click(submitButton);
    
    await waitFor(() => {
      expect(mockOnSearch).toHaveBeenCalledWith('test query');
    });
  });
});
```

## Code Review Process

### Before Requesting Review

**Self-Review Checklist:**
- [ ] Code follows style guidelines
- [ ] All tests pass
- [ ] No console errors in development
- [ ] Responsive design tested on mobile
- [ ] Accessibility considerations addressed
- [ ] Performance impact considered

### Review Guidelines

**For Reviewers:**
- Focus on logic, not style (handled by linting)
- Check for security vulnerabilities
- Ensure tests cover new functionality
- Verify accessibility compliance
- Consider performance implications

**For Authors:**
- Respond to all feedback
- Make requested changes in new commits
- Ask for clarification when needed
- Update PR description if scope changes

### Review Checklist

#### Functionality
- [ ] Feature works as described
- [ ] Edge cases handled appropriately
- [ ] Error handling implemented
- [ ] No breaking changes to existing features

#### Code Quality
- [ ] Code is readable and well-documented
- [ ] No code duplication
- [ ] Proper separation of concerns
- [ ] Consistent with project patterns

#### Testing
- [ ] New functionality has tests
- [ ] Tests cover edge cases
- [ ] Existing tests still pass
- [ ] Manual testing performed

#### Performance
- [ ] No unnecessary re-renders
- [ ] Database queries optimized
- [ ] Large datasets handled efficiently
- [ ] Bundle size impact minimal

## Release Process

### Versioning

We follow **Semantic Versioning** (semver):
- **Major** (1.0.0): Breaking changes
- **Minor** (0.1.0): New features (backward compatible)
- **Patch** (0.0.1): Bug fixes (backward compatible)

### Release Steps

#### 1. Pre-Release
```bash
# Ensure main is up to date
git checkout main
git pull origin main

# Run full test suite
yarn test:ci

# Build and verify
yarn build
```

#### 2. Version Update
```bash
# Update version in package.json
npm version patch  # or minor/major

# This creates a git tag automatically
git push origin main --tags
```

#### 3. Deploy
```bash
# Deploy to staging first
yarn deploy:staging

# Test staging environment
# Run smoke tests

# Deploy to production
yarn deploy:prod
```

#### 4. Post-Release
- Monitor error logs
- Check key metrics (search success rate, response times)
- Update documentation if needed
- Create GitHub release notes

### Hotfix Process

For critical production issues:

```bash
# Create hotfix branch from main
git checkout -b hotfix/critical-fix main

# Make minimal fix
# Test thoroughly

# Create PR with "HOTFIX" label
# Get immediate review
# Deploy directly to production after merge
```

## Common Tasks

### Adding a New Feature

1. **Planning**
   - Create GitHub issue
   - Define acceptance criteria
   - Consider impact on existing features

2. **Development**
   - Create feature branch
   - Implement with tests
   - Update documentation

3. **Integration**
   - Create pull request
   - Address review feedback
   - Merge and deploy

### Debugging Issues

#### Development Debugging
```bash
# Check server logs
yarn dev  # Watch for console output

# Database debugging
psql -U firewallcafe -d firewallcafe
SELECT * FROM searches ORDER BY timestamp DESC LIMIT 5;

# API debugging
curl http://localhost:8080/api/searches?limit=1
```

#### Production Debugging
- Check Google Cloud Platform logs
- Monitor database performance
- Use error tracking service
- Check API rate limits

### Performance Optimization

#### Frontend Optimization
- Use React.memo for expensive components
- Implement virtualization for long lists
- Lazy load images and non-critical components
- Optimize bundle size with code splitting

#### Backend Optimization
- Implement database query caching
- Optimize SQL queries with indexes
- Use connection pooling
- Monitor API response times

## Resources

### Documentation
- [React Documentation](https://react.dev)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Express.js Guide](https://expressjs.com)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)

### Tools
- [GitHub CLI](https://cli.github.com) for PR management
- [VS Code Extensions](../SETUP.md#vs-code-extensions) 
- [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools)

### Style Guides
- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [React Best Practices](https://react.dev/learn)
- [Tailwind CSS Best Practices](https://tailwindcss.com/docs/reusing-styles)

---

*Need help or have questions? Check the [Contributing Guidelines](./CONTRIBUTING.md) or create an issue on GitHub.*