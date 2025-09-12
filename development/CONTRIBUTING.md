# Contributing to Firewall Cafe

Thank you for your interest in contributing to Firewall Cafe! This guide will help you get started with contributing to our project that helps researchers, journalists, and educators understand internet censorship.

## Table of Contents
- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Development Workflow](#development-workflow)
- [Contribution Guidelines](#contribution-guidelines)
- [Review Process](#review-process)
- [Community](#community)

## Code of Conduct

### Our Pledge
We pledge to make participation in our project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, sex characteristics, gender identity and expression, level of experience, education, socio-economic status, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Expected Behavior
- Use welcoming and inclusive language
- Be respectful of differing viewpoints and experiences
- Gracefully accept constructive criticism
- Focus on what is best for the community
- Show empathy towards other community members

### Unacceptable Behavior
- Trolling, insulting/derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information without explicit permission
- Other conduct which could reasonably be considered inappropriate in a professional setting

## Getting Started

### Prerequisites
Before contributing, ensure you have:
- Node.js v20.x installed
- Yarn package manager
- PostgreSQL database
- Git configured with your GitHub account
- Read our [Architecture Overview](../ARCHITECTURE.md)

### Initial Setup
1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/your-username/firewall-cafe.git
   cd firewall-cafe
   ```
3. **Set up development environment** following our [Setup Guide](../SETUP.md)
4. **Add upstream remote**:
   ```bash
   git remote add upstream https://github.com/FIREWALL-cafe/firewall-cafe.git
   ```

## How to Contribute

### Types of Contributions We Welcome

#### 🐛 Bug Reports
Help us identify and fix issues:
- Search existing issues first
- Use our bug report template
- Include steps to reproduce
- Provide environment details

#### 💡 Feature Requests
Propose new functionality:
- Check the [roadmap](../product/roadmap.md) first
- Open a discussion issue
- Explain the use case and benefits
- Consider implementation complexity

#### 🔧 Code Contributions
Direct code improvements:
- Bug fixes
- Feature implementations
- Performance improvements
- Test coverage improvements
- Documentation updates

#### 📚 Documentation
Help improve our docs:
- Fix typos or unclear sections
- Add missing documentation
- Improve code examples
- Update outdated information

### What We Need Help With

#### High Priority
- **Database Performance**: Query optimization and indexing
- **Search API Integration**: Improving Google/Baidu search accuracy
- **Geographic Visualization**: Enhanced heatmap features
- **Mobile Experience**: Responsive design improvements
- **Accessibility**: WCAG compliance and screen reader support

#### Medium Priority
- **Testing**: Unit and integration test coverage
- **Error Handling**: Better error messages and recovery
- **Performance**: Frontend optimization and caching
- **Internationalization**: Multi-language support beyond EN/ZH
- **Analytics**: Advanced data visualization features

#### Welcome Contributions
- **Bug Fixes**: Any size, especially with tests
- **Documentation**: Improvements, examples, translations
- **Code Cleanup**: Refactoring, removing dead code
- **Developer Experience**: Build tools, linting, CI/CD

## Development Workflow

### Before You Start
1. **Check existing issues** and pull requests
2. **Create or comment** on an issue to discuss your approach
3. **Get assignment** or approval from maintainers for large changes
4. **Read relevant documentation** in `/docs/`

### Making Changes

#### 1. Create a Branch
```bash
# Sync with upstream
git checkout main
git pull upstream main

# Create feature branch
git checkout -b feature/your-feature-name
```

#### 2. Make Your Changes
- Follow our [Development Workflow](./WORKFLOW.md)
- Write clear, focused commits
- Include tests for new functionality
- Update documentation as needed

#### 3. Test Your Changes
```bash
# Run linting
yarn lint:fix

# Run tests
yarn test:ci

# Manual testing
yarn dev
# Test in browser at localhost:3000
```

#### 4. Commit Your Changes
```bash
# Stage changes
git add .

# Commit with conventional message
git commit -m "feat(search): add real-time comparison feature"
```

### Commit Message Format
We use [Conventional Commits](https://www.conventionalcommits.org/) for clear history:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix  
- `docs`: Documentation only changes
- `style`: Changes that don't affect meaning (formatting, etc.)
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `test`: Adding missing tests or correcting existing tests
- `chore`: Changes to build process or auxiliary tools

**Examples:**
```bash
feat(api): add pagination support to search endpoints
fix(heatmap): resolve country selection bug
docs(setup): update database configuration steps
test(search): add integration tests for comparison feature
```

## Contribution Guidelines

### Code Quality Standards

#### JavaScript/React Code
```javascript
// ✅ Good: Clear, functional component with proper hooks
import React, { useState, useEffect } from 'react';

function SearchComparison({ query, onResultsChange }) {
  const [results, setResults] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    if (query) {
      fetchComparison(query);
    }
  }, [query]);
  
  const fetchComparison = async (searchQuery) => {
    setLoading(true);
    setError(null);
    
    try {
      const data = await api.compareSearch(searchQuery);
      setResults(data);
      onResultsChange?.(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  if (error) return <ErrorDisplay error={error} />;
  if (loading) return <LoadingSpinner />;
  
  return (
    <div className="search-comparison">
      {/* Component JSX */}
    </div>
  );
}

export default SearchComparison;
```

#### CSS/Styling
- Use Tailwind CSS classes for consistent styling
- Follow responsive design patterns (mobile-first)
- Maintain accessibility with proper focus states
- Use semantic HTML elements

```javascript
// ✅ Good: Organized Tailwind classes
<button className="
  px-4 py-2 
  bg-blue-600 hover:bg-blue-700 active:bg-blue-800
  text-white font-medium rounded-lg
  focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
  disabled:opacity-50 disabled:cursor-not-allowed
  transition-colors duration-200
">
```

### Testing Requirements

#### Required Tests
- **Unit Tests**: For utility functions and complex logic
- **Component Tests**: For React components with user interactions
- **Integration Tests**: For API endpoints and data flow
- **Manual Testing**: Browser testing across different screen sizes

#### Test Example
```javascript
// SearchForm.test.jsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import SearchForm from './SearchForm';

describe('SearchForm', () => {
  const mockOnSearch = jest.fn();
  
  beforeEach(() => {
    mockOnSearch.mockClear();
  });
  
  it('submits search with valid input', async () => {
    render(<SearchForm onSearch={mockOnSearch} />);
    
    const input = screen.getByLabelText(/search term/i);
    const button = screen.getByRole('button', { name: /search/i });
    
    fireEvent.change(input, { target: { value: 'climate change' } });
    fireEvent.click(button);
    
    await waitFor(() => {
      expect(mockOnSearch).toHaveBeenCalledWith({
        query: 'climate change',
        language: 'auto'
      });
    });
  });
  
  it('shows validation error for empty input', () => {
    render(<SearchForm onSearch={mockOnSearch} />);
    
    const button = screen.getByRole('button', { name: /search/i });
    fireEvent.click(button);
    
    expect(screen.getByText(/search term is required/i)).toBeInTheDocument();
    expect(mockOnSearch).not.toHaveBeenCalled();
  });
});
```

### Documentation Requirements

#### Code Documentation
- **JSDoc comments** for complex functions
- **PropTypes** or TypeScript for component props
- **README updates** for new features
- **API documentation** for new endpoints

```javascript
/**
 * Compares search results between Google and Baidu
 * @param {string} query - The search term to compare
 * @param {Object} options - Search options
 * @param {string} options.language - Language code (en, zh, auto)
 * @param {string} options.location - Geographic location filter
 * @returns {Promise<Object>} Comparison results with Google and Baidu data
 */
async function compareSearchResults(query, options = {}) {
  // Implementation
}
```

### Accessibility Guidelines

#### Requirements
- **Semantic HTML**: Use proper heading hierarchy, landmarks
- **Keyboard Navigation**: All interactive elements accessible via keyboard
- **Screen Readers**: ARIA labels, descriptions, live regions
- **Color Contrast**: Meet WCAG AA standards (4.5:1 minimum)
- **Focus Management**: Visible focus indicators, logical tab order

#### Example
```javascript
// ✅ Accessible component
function SearchButton({ onClick, loading, disabled }) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      aria-label={loading ? 'Searching...' : 'Search'}
      aria-describedby="search-help"
      className="
        px-4 py-2 bg-blue-600 text-white rounded-lg
        focus:outline-none focus:ring-2 focus:ring-blue-500
        disabled:opacity-50 disabled:cursor-not-allowed
      "
    >
      {loading ? (
        <>
          <span className="sr-only">Searching...</span>
          <LoadingSpinner size="small" />
        </>
      ) : (
        'Search'
      )}
    </button>
  );
}
```

## Review Process

### Pull Request Guidelines

#### Before Submitting
- [ ] Code follows style guidelines
- [ ] All tests pass locally
- [ ] Documentation updated
- [ ] Manual testing completed
- [ ] Commit messages follow convention
- [ ] Branch is up to date with main

#### PR Description Template
```markdown
## Description
Brief description of changes and motivation.

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to change)
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass  
- [ ] Manual testing performed
- [ ] Accessibility testing completed

## Screenshots (if applicable)
[Add screenshots of UI changes]

## Checklist
- [ ] My code follows the project style guidelines
- [ ] I have performed a self-review of my code
- [ ] I have commented my code, particularly hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
```

### Review Process Steps

#### 1. Automated Checks
- Linting and formatting (ESLint, Prettier)
- Test suite execution
- Build verification
- Security scanning

#### 2. Human Review
- **Code Quality**: Logic, performance, maintainability
- **Functionality**: Feature works as intended
- **Design**: UI/UX consistency and accessibility
- **Documentation**: Clear and complete

#### 3. Feedback Response
- Address all reviewer comments
- Make requested changes in new commits
- Re-request review after changes
- Discussion and collaboration on solutions

### Review Timeline
- **Small fixes**: 1-2 days
- **Features**: 3-5 days
- **Large changes**: 1-2 weeks

Maintainers will provide initial feedback within 2 business days.

## Community

### Communication Channels
- **GitHub Issues**: Bug reports, feature requests, discussions
- **GitHub Discussions**: General community conversations
- **Email**: info@firewallcafe.com for sensitive issues

### Getting Help
1. **Search existing issues** and documentation
2. **Create a new issue** with detailed information
3. **Join community discussions** for broader topics
4. **Contact maintainers** for urgent or sensitive matters

### Recognition
We recognize contributors through:
- GitHub contributor graphs
- Release notes acknowledgments
- Community spotlight features
- Maintainer nominations for significant contributors

### Maintainer Responsibilities
Current maintainers commit to:
- Respond to issues and PRs within 2 business days
- Provide constructive feedback and guidance
- Maintain project roadmap and priorities
- Foster inclusive, welcoming community

## License
By contributing to Firewall Cafe, you agree that your contributions will be licensed under the same [MIT License](../../LICENSE) that covers the project.

---

## Quick Reference

### Useful Commands
```bash
# Setup and development
yarn install          # Install dependencies
yarn dev              # Start development server
yarn test:ci          # Run full test suite
yarn lint:fix         # Fix linting issues
yarn format           # Format code

# Git workflow
git checkout -b feature/my-feature  # Create feature branch
git commit -m "feat: add feature"   # Commit changes
git push origin feature/my-feature  # Push to GitHub
```

### Key Documentation
- [Setup Guide](../SETUP.md) - Development environment
- [Architecture](../ARCHITECTURE.md) - System overview  
- [Workflow](./WORKFLOW.md) - Development process
- [API Docs](../api/README.md) - Backend API reference
- [Components](../frontend/COMPONENTS.md) - Frontend patterns

### Need Help?
- 📖 **Documentation**: Check `/docs/` directory
- 🐛 **Bugs**: Create GitHub issue with reproduction steps
- 💡 **Features**: Start with GitHub discussion  
- ❓ **Questions**: Check existing issues or create new one
- 🚨 **Security**: Email info@firewallcafe.com

Thank you for contributing to Firewall Cafe! Your efforts help make internet censorship research more accessible to researchers, journalists, and educators worldwide. 🌍