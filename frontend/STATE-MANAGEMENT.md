# State Management

> Last Updated: 2025-01-12
> Architecture: React Context + Component State
> Primary Files: [`firewall-client-server/client/src/SearchContext.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/SearchContext.js)

## Overview

Firewall Cafe uses a **minimal state management approach** combining React Context for global state with component-level state management. This lightweight architecture is well-suited for the application's moderate complexity.

## Architecture

### Global State (React Context)

**SearchContext** 
- **Purpose**: Share search results and metadata across components
- **Scope**: Search comparison data, translations, image results
- **Usage**: Provider wraps main application components
- **Implementation**: [`client/src/SearchContext.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/SearchContext.js)

### Component-Level State

**FilterControls Component** 
- **Purpose**: Manage complex filtering UI state
- **Pattern**: Multiple useState hooks for different filter types
- **Implementation**: [`client/src/components/FilterControls.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/FilterControls.jsx)

## State Patterns

### Search Flow State

**Search Input Component**
- **File**: [`client/src/components/SearchInput.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/SearchInput.jsx)
- **State Pattern**: Context consumption with local loading state
- **Flow**: User input → API call → Context update → Component re-render

### Filter State Management

**Complex Filter Logic:**
- **Implementation**: [`client/src/components/FilterControls.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/FilterControls.jsx)
- **Features**: 
  - Hierarchical geographic filters (country → city)
  - Date range validation
  - Vote category multi-select
  - Dynamic city list based on country selection

### Pagination State

**Archive Component**
- **File**: [`client/src/components/SearchArchive.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/SearchArchive.jsx)
- **State Management**: 
  - Search results array
  - Pagination metadata (current page, total pages, total results)
  - Loading state for async operations

## Data Flow

### Search Comparison Flow

1. **User Input** → SearchInput component state
2. **API Call** → Loading state management
3. **Results** → SearchContext global state
4. **Display** → Search component consumes context

**Flow Diagram**: User Input → SearchInput State → API Call → SearchContext Update → Component Re-render

### Filter Flow

1. **Filter Changes** → FilterControls component state
2. **Validation** → Date range, geographic hierarchy
3. **API Call** → SearchArchive with filter parameters
4. **Results Update** → Archive component state

**Flow Diagram**: Filter Selection → State Update → Validation → API Call → Results Update

## State Synchronization

### URL State Sync

**Route-based State** 
- **Implementation**: Dashboard component with URL parameter sync
- **Pattern**: React Router's useSearchParams hook
- **Example**: Dashboard tabs stored in URL for shareable state
- **File**: [`client/src/components/Dashboard.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/Dashboard.jsx)

### Cross-Component Communication

**Voting State Update:**
- **Pattern**: Local state with optimistic updates
- **Flow**: Submit vote → Update local state → Fetch new counts
- **Implementation**: Voting components in search results

## Performance Optimizations

### State Updates

**Performance Patterns:**
- **Batched Updates**: React 18 automatic batching for multiple state changes
- **Memoization**: useMemo for expensive filter operations
- **Debounced Search**: 500ms delay for search input
- **Implementation Examples**: Throughout component files in `client/src/components/`

## Error State Management

### Error State Management

**Error Handling Patterns:**
- **Error Boundaries**: Component-level error catching
- **API Error States**: Loading, error, and data states for API calls
- **Retry Logic**: User-triggered retry mechanisms
- **Implementation**: Custom hooks and error boundary components

## Testing State Management

### Testing State Management

**Testing Patterns:**
- **Context Testing**: Wrapper components for test isolation
- **Hook Testing**: renderHook from React Testing Library
- **State Updates**: act() wrapper for state changes
- **Test Files**: Located in `client/src/__tests__/`

## Best Practices

### State Structure

1. **Keep state flat** - Avoid deep nesting
2. **Separate concerns** - Different components for different state domains
3. **Minimize global state** - Use context sparingly
4. **Predictable updates** - Use functional updates for complex state

### Component Organization

1. **Container/Presentation** - Separate state logic from UI
2. **Custom hooks** - Extract reusable state logic
3. **Error boundaries** - Wrap stateful components
4. **Loading states** - Always handle async operations

---

*For component structure details, see [Component Library](./COMPONENTS.md)*
*For routing implementation, see [Routing](./ROUTING.md)*