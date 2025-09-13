# Routing

> Last Updated: 2025-01-12  
> Framework: React Router v6.27.0
> Configuration: `/client/src/index.js`

## Overview

Firewall Cafe uses **React Router v6** for client-side routing with a clean, hierarchical structure. The routing is designed around the main user flows: search comparison, archive browsing, and analytics dashboard.

## Route Structure

### Main Application Routes

**Root Configuration** (`/client/src/index.js`):

```javascript
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Layout from './components/Layout';

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Search />} />
          <Route path="search" element={<Search />} />
          <Route path="archive" element={<SearchArchive />} />
          <Route path="dashboard" element={<Dashboard />} />
          <Route path="timeline" element={<Timeline />} />
          <Route path="about" element={<About />} />
          <Route path="terms" element={<TermsAndConditions />} />
          <Route path="privacy" element={<Privacy />} />
          <Route path="contact" element={<Contact />} />
          <Route path="demo" element={<Demo />} />
          <Route path="research" element={<Research />} />
          <Route path="education" element={<Education />} />
          <Route path="*" element={<NotFound />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
};
```

### Route Hierarchy

```
/                    → Search (home page)
├── /search          → Search comparison interface
├── /archive         → Historical search browsing
├── /dashboard       → Analytics dashboard
│   ├── ?tab=overview     → Dashboard overview tab
│   ├── ?tab=geographic   → Geographic analytics
│   └── ?tab=votes        → Vote analytics
├── /timeline        → Temporal search visualization
├── /about          → About Firewall Cafe
├── /terms          → Terms and conditions
├── /privacy        → Privacy policy
├── /contact        → Contact information
├── /demo           → Demo mode
├── /research       → Research resources
├── /education      → Educational content
└── /*              → 404 Not Found
```

## Layout Component

### Nested Route Structure

**Layout Component** (`/client/src/components/Layout.jsx`):

```javascript
import { Outlet, useLocation } from 'react-router-dom';
import Navigation from './Navigation';
import Footer from './Footer';

const Layout = () => {
  const location = useLocation();
  
  return (
    <div className="min-h-screen flex flex-col">
      <Navigation currentPath={location.pathname} />
      
      <main className="flex-1">
        <Outlet />
      </main>
      
      <Footer />
    </div>
  );
};
```

**Benefits:**
- Consistent navigation across all routes
- Shared layout and styling
- Efficient re-rendering (only Outlet content changes)

## Navigation Implementation

### Dynamic Navigation

**Navigation Component** (`/client/src/components/Navigation.jsx`):

```javascript
import { Link, useLocation } from 'react-router-dom';

const Navigation = () => {
  const location = useLocation();
  
  const navigationItems = [
    { path: '/', label: 'Search', exact: true },
    { path: '/archive', label: 'Archive' },
    { path: '/dashboard', label: 'Dashboard' },
    { path: '/timeline', label: 'Timeline' },
    { path: '/about', label: 'About' }
  ];
  
  const isActive = (path, exact = false) => {
    if (exact) {
      return location.pathname === path;
    }
    return location.pathname.startsWith(path);
  };
  
  return (
    <nav className="bg-white border-b border-gray-200">
      <div className="max-w-7xl mx-auto px-4">
        <div className="flex justify-between items-center h-16">
          <Link to="/" className="text-xl font-bold">
            Firewall Cafe
          </Link>
          
          <div className="flex space-x-8">
            {navigationItems.map(({ path, label, exact }) => (
              <Link
                key={path}
                to={path}
                className={`${
                  isActive(path, exact)
                    ? 'text-blue-600 border-b-2 border-blue-600'
                    : 'text-gray-700 hover:text-blue-600'
                } px-3 py-2 text-sm font-medium`}
              >
                {label}
              </Link>
            ))}
          </div>
        </div>
      </div>
    </nav>
  );
};
```

## Route-Specific Components

### Search Route (`/`, `/search`)

**Primary User Interface:**

```javascript
const Search = () => {
  const [searchResults, setSearchResults] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  
  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      <SearchInput onSearch={handleSearch} />
      
      {isLoading && <LoadingSpinner />}
      
      {searchResults && (
        <SearchResults 
          googleResults={searchResults.google}
          baiduResults={searchResults.baidu}
          searchId={searchResults.search_id}
        />
      )}
    </div>
  );
};
```

### Archive Route (`/archive`)

**Historical Search Browsing:**

```javascript
const SearchArchive = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  const [searches, setSearches] = useState([]);
  const [filters, setFilters] = useState({
    page: parseInt(searchParams.get('page')) || 1,
    countries: searchParams.getAll('country') || [],
    startDate: searchParams.get('start_date') || '',
    endDate: searchParams.get('end_date') || ''
  });
  
  // Sync URL with filter state
  useEffect(() => {
    const params = new URLSearchParams();
    if (filters.page > 1) params.set('page', filters.page);
    if (filters.countries.length) {
      filters.countries.forEach(country => params.append('country', country));
    }
    if (filters.startDate) params.set('start_date', filters.startDate);
    if (filters.endDate) params.set('end_date', filters.endDate);
    
    setSearchParams(params);
  }, [filters, setSearchParams]);
  
  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      <FilterControls filters={filters} onChange={setFilters} />
      <SearchGrid searches={searches} />
      <Pagination 
        currentPage={filters.page}
        totalPages={totalPages}
        onPageChange={(page) => setFilters(prev => ({ ...prev, page }))}
      />
    </div>
  );
};
```

### Dashboard Route (`/dashboard`)

**Analytics with Tab Navigation:**

```javascript
const Dashboard = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  const activeTab = searchParams.get('tab') || 'overview';
  
  const tabs = [
    { id: 'overview', label: 'Overview' },
    { id: 'geographic', label: 'Geographic' },
    { id: 'votes', label: 'Votes' },
    { id: 'trends', label: 'Trends' }
  ];
  
  const handleTabChange = (tabId) => {
    setSearchParams({ tab: tabId });
  };
  
  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      <div className="border-b border-gray-200 mb-8">
        <nav className="-mb-px flex space-x-8">
          {tabs.map(({ id, label }) => (
            <button
              key={id}
              onClick={() => handleTabChange(id)}
              className={`${
                activeTab === id
                  ? 'border-blue-500 text-blue-600'
                  : 'border-transparent text-gray-500 hover:text-gray-700'
              } whitespace-nowrap py-2 px-1 border-b-2 font-medium text-sm`}
            >
              {label}
            </button>
          ))}
        </nav>
      </div>
      
      {activeTab === 'overview' && <OverviewDashboard />}
      {activeTab === 'geographic' && <GeographicDashboard />}
      {activeTab === 'votes' && <VotesDashboard />}
      {activeTab === 'trends' && <TrendsDashboard />}
    </div>
  );
};
```

## URL Parameter Management

### Query String Handling

**Search Parameters Hook:**

```javascript
const useFilters = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const filters = {
    countries: searchParams.getAll('country'),
    cities: searchParams.getAll('city'),
    startDate: searchParams.get('start_date'),
    endDate: searchParams.get('end_date'),
    page: parseInt(searchParams.get('page')) || 1,
    voteCategories: searchParams.getAll('vote').map(Number)
  };
  
  const updateFilters = (newFilters) => {
    const params = new URLSearchParams();
    
    // Handle arrays
    newFilters.countries?.forEach(c => params.append('country', c));
    newFilters.cities?.forEach(c => params.append('city', c));
    newFilters.voteCategories?.forEach(v => params.append('vote', v));
    
    // Handle single values
    if (newFilters.startDate) params.set('start_date', newFilters.startDate);
    if (newFilters.endDate) params.set('end_date', newFilters.endDate);
    if (newFilters.page && newFilters.page > 1) params.set('page', newFilters.page);
    
    setSearchParams(params);
  };
  
  return { filters, updateFilters };
};
```

### Deep Linking

**Shareable URLs:**

```javascript
// Archive with filters
// /archive?country=China&country=USA&start_date=2025-01-01&page=2

// Dashboard with specific tab
// /dashboard?tab=geographic

// Search with preset query
// /search?q=censorship&auto=true
```

## Route Guards & Authentication

### Protected Routes (Future Implementation)

```javascript
const ProtectedRoute = ({ children, requiredRole }) => {
  const { user, isLoading } = useAuth();
  const location = useLocation();
  
  if (isLoading) return <LoadingSpinner />;
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }
  
  return children;
};

// Usage
<Route 
  path="/admin" 
  element={
    <ProtectedRoute requiredRole="admin">
      <AdminDashboard />
    </ProtectedRoute>
  } 
/>
```

## Error Boundaries

### Route-Level Error Handling

```javascript
const RouteErrorBoundary = ({ children }) => {
  const [hasError, setHasError] = useState(false);
  const location = useLocation();
  
  useEffect(() => {
    if (hasError) setHasError(false);
  }, [location.pathname]);
  
  if (hasError) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="text-center">
          <h1 className="text-2xl font-bold text-gray-900 mb-4">
            Oops! Something went wrong
          </h1>
          <p className="text-gray-600 mb-4">
            There was an error loading this page.
          </p>
          <Link 
            to="/" 
            className="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
          >
            Go Home
          </Link>
        </div>
      </div>
    );
  }
  
  return children;
};
```

## Performance Optimizations

### Lazy Loading

```javascript
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./components/Dashboard'));
const Timeline = lazy(() => import('./components/Timeline'));

// In routes
<Route 
  path="/dashboard" 
  element={
    <Suspense fallback={<LoadingSpinner />}>
      <Dashboard />
    </Suspense>
  } 
/>
```

### Route Preloading

```javascript
const preloadRoute = (routeComponent) => {
  routeComponent();
};

// Preload on hover
<Link 
  to="/dashboard"
  onMouseEnter={() => preloadRoute(() => import('./Dashboard'))}
>
  Dashboard
</Link>
```

## Testing Routes

### Route Testing

```javascript
import { render, screen } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import App from './App';

test('renders search page on root route', () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );
  
  expect(screen.getByText('Search Comparison')).toBeInTheDocument();
});

test('renders 404 on unknown route', () => {
  render(
    <MemoryRouter initialEntries={['/unknown']}>
      <App />
    </MemoryRouter>
  );
  
  expect(screen.getByText('Page Not Found')).toBeInTheDocument();
});
```

### Navigation Testing

```javascript
import { fireEvent } from '@testing-library/react';

test('navigation updates active state', () => {
  render(
    <MemoryRouter>
      <Navigation />
    </MemoryRouter>
  );
  
  const archiveLink = screen.getByText('Archive');
  fireEvent.click(archiveLink);
  
  expect(archiveLink).toHaveClass('text-blue-600');
});
```

## Best Practices

### URL Design
1. **Descriptive paths** - Clear, readable URLs
2. **Consistent patterns** - Similar routes follow same structure
3. **State preservation** - Important filters in URL
4. **SEO friendly** - Meaningful route names

### Component Organization
1. **Route components** - One component per route
2. **Nested layouts** - Shared UI through Layout
3. **Error boundaries** - Graceful error handling
4. **Loading states** - Proper async handling

### Performance
1. **Lazy loading** - Split code by routes
2. **Prefetching** - Preload likely next routes
3. **State cleanup** - Clear component state on route change
4. **Memory management** - Prevent memory leaks

---

*For component structure details, see [Component Library](./COMPONENTS.md)*
*For state management across routes, see [State Management](./STATE-MANAGEMENT.md)*