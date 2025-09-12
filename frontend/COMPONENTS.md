# Frontend Components Guide

This guide covers the React component architecture, state management patterns, and frontend development practices for Firewall Cafe.

## Table of Contents
- [Component Architecture](#component-architecture)
- [State Management](#state-management)  
- [Key Components](#key-components)
- [Styling Guidelines](#styling-guidelines)
- [Performance Patterns](#performance-patterns)
- [Testing Components](#testing-components)

## Component Architecture

### File Structure
```
src/
├── components/           # All React components
│   ├── search/          # Search-related components
│   ├── analytics/       # Dashboard and analytics
│   ├── maps/           # Geographic visualization
│   ├── voting/         # Voting interface
│   └── common/         # Shared/reusable components
├── contexts/           # React Context providers
├── hooks/              # Custom React hooks
├── utils/              # Helper functions
├── constants/          # Configuration and mappings
└── assets/            # Images, icons, fonts
```

### Component Categories

#### 1. **Page Components** (Route-level)
- Handle routing and high-level state
- Connect to context providers
- Coordinate multiple feature components

#### 2. **Feature Components** (Domain-specific)
- Search comparison functionality
- Analytics dashboards
- Geographic heatmaps
- Voting interfaces

#### 3. **Common Components** (Reusable)
- Buttons, inputs, modals
- Loading states, error boundaries
- Navigation, layout components

## State Management

### React Context Pattern

The application uses React Context for global state management:

#### ApiContext
```javascript
// contexts/ApiContext.js
import { createContext } from 'react';

const ApiContext = createContext();

// API functions for data fetching
const searchArchive = async (options) => {
  const response = await fetch(`/searches?${querystring.stringify(options)}`);
  return response.json();
};

export { ApiContext, searchArchive };
```

#### Usage in Components
```javascript
import { useContext, useEffect, useState } from 'react';
import { ApiContext } from '../contexts/ApiContext';

function SearchResults() {
  const [searches, setSearches] = useState([]);
  const { searchArchive } = useContext(ApiContext);
  
  useEffect(() => {
    const fetchData = async () => {
      const results = await searchArchive({ limit: 10 });
      setSearches(results);
    };
    fetchData();
  }, []);
  
  return (
    <div className="search-results">
      {searches.map(search => (
        <SearchCard key={search.searchId} search={search} />
      ))}
    </div>
  );
}
```

### Local State Patterns

#### Form State Management
```javascript
function SearchForm({ onSearch }) {
  const [formData, setFormData] = useState({
    query: '',
    language: 'auto',
    location: ''
  });
  
  const handleChange = (field) => (event) => {
    setFormData(prev => ({
      ...prev,
      [field]: event.target.value
    }));
  };
  
  const handleSubmit = (event) => {
    event.preventDefault();
    onSearch(formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={formData.query}
        onChange={handleChange('query')}
        placeholder="Enter search term..."
      />
      {/* More form fields */}
    </form>
  );
}
```

## Key Components

### 1. Search Components

#### SearchForm
**Purpose**: Handles user input for search queries
**Location**: `components/search/SearchForm.jsx`

```javascript
function SearchForm({ onSearch, loading = false }) {
  const [query, setQuery] = useState('');
  const [language, setLanguage] = useState('auto');
  
  const handleSubmit = async (event) => {
    event.preventDefault();
    if (query.trim()) {
      await onSearch({ query: query.trim(), language });
      setQuery(''); // Clear after search
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="search-form">
      <div className="flex gap-4">
        <input
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Enter search term..."
          className="flex-1 px-4 py-2 border rounded-lg"
          disabled={loading}
        />
        <select
          value={language}
          onChange={(e) => setLanguage(e.target.value)}
          className="px-3 py-2 border rounded-lg"
        >
          <option value="auto">Auto-detect</option>
          <option value="en">English</option>
          <option value="zh">Chinese</option>
        </select>
        <button
          type="submit"
          disabled={loading || !query.trim()}
          className="px-6 py-2 bg-blue-600 text-white rounded-lg disabled:opacity-50"
        >
          {loading ? 'Searching...' : 'Search'}
        </button>
      </div>
    </form>
  );
}
```

#### SearchComparison
**Purpose**: Displays side-by-side Google vs Baidu results
**Location**: `components/search/SearchComparison.jsx`

```javascript
function SearchComparison({ searchData }) {
  const { googleResults, baiduResults, translation } = searchData;
  
  return (
    <div className="search-comparison grid grid-cols-1 md:grid-cols-2 gap-6">
      {/* Google Results */}
      <div className="search-column">
        <h2 className="text-xl font-bold mb-4 flex items-center">
          <img src="/google-icon.png" alt="Google" className="w-6 h-6 mr-2" />
          Google Results
        </h2>
        <SearchResults results={googleResults} engine="google" />
      </div>
      
      {/* Baidu Results */}
      <div className="search-column">
        <h2 className="text-xl font-bold mb-4 flex items-center">
          <img src="/baidu-icon.png" alt="Baidu" className="w-6 h-6 mr-2" />
          Baidu Results
        </h2>
        <SearchResults results={baiduResults} engine="baidu" />
      </div>
      
      {/* Translation Info */}
      {translation && (
        <div className="col-span-1 md:col-span-2 mt-4 p-4 bg-gray-100 rounded-lg">
          <p className="text-sm">
            <strong>Translation:</strong> "{translation.original}" → "{translation.translated}"
          </p>
        </div>
      )}
    </div>
  );
}
```

### 2. Analytics Components

#### AnalyticsDashboard
**Purpose**: Main analytics overview with charts and metrics
**Location**: `components/analytics/AnalyticsDashboard.jsx`

```javascript
import { Chart as ChartJS } from 'chart.js/auto';
import { Bar, Line, Doughnut } from 'react-chartjs-2';

function AnalyticsDashboard() {
  const [analytics, setAnalytics] = useState(null);
  const [timeframe, setTimeframe] = useState('30d');
  
  useEffect(() => {
    const fetchAnalytics = async () => {
      const [searches, votes, geographic] = await Promise.all([
        fetch(`/api/analytics/searches?timeframe=${timeframe}`).then(r => r.json()),
        fetch(`/api/analytics/votes?timeframe=${timeframe}`).then(r => r.json()),
        fetch(`/api/analytics/geographic?timeframe=${timeframe}`).then(r => r.json())
      ]);
      
      setAnalytics({ searches, votes, geographic });
    };
    
    fetchAnalytics();
  }, [timeframe]);
  
  if (!analytics) return <LoadingSpinner />;
  
  return (
    <div className="analytics-dashboard space-y-8">
      {/* Time Range Selector */}
      <div className="flex justify-between items-center">
        <h1 className="text-2xl font-bold">Analytics Dashboard</h1>
        <select
          value={timeframe}
          onChange={(e) => setTimeframe(e.target.value)}
          className="px-3 py-2 border rounded-lg"
        >
          <option value="7d">Last 7 days</option>
          <option value="30d">Last 30 days</option>
          <option value="90d">Last 90 days</option>
        </select>
      </div>
      
      {/* Metrics Cards */}
      <div className="grid grid-cols-1 md:grid-cols-4 gap-6">
        <MetricCard title="Total Searches" value={analytics.searches.total} />
        <MetricCard title="Total Votes" value={analytics.votes.total} />
        <MetricCard title="Countries" value={analytics.geographic.countryCount} />
        <MetricCard title="Languages" value={analytics.searches.languageCount} />
      </div>
      
      {/* Charts */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <ChartCard title="Search Trends">
          <Line data={analytics.searches.timeline} options={chartOptions} />
        </ChartCard>
        
        <ChartCard title="Vote Distribution">  
          <Doughnut data={analytics.votes.distribution} options={chartOptions} />
        </ChartCard>
      </div>
    </div>
  );
}
```

### 3. Geographic Components

#### GeographicHeatmap
**Purpose**: Interactive world map showing search density
**Location**: `components/maps/GeographicHeatmap.jsx`

```javascript
import { ComposableMap, Geographies, Geography } from 'react-simple-maps';
import { scaleLinear } from 'd3-scale';

function GeographicHeatmap({ data, onCountryClick }) {
  const colorScale = scaleLinear()
    .domain([0, Math.max(...data.map(d => d.count))])
    .range(['#e6f3ff', '#0066cc']);
    
  const getCountryValue = (countryCode) => {
    const country = data.find(d => d.countryCode === countryCode);
    return country ? country.count : 0;
  };
  
  return (
    <div className="geographic-heatmap">
      <ComposableMap>
        <Geographies geography="/world-atlas.json">
          {({ geographies }) =>
            geographies.map(geo => {
              const value = getCountryValue(geo.properties.ISO_A2);
              return (
                <Geography
                  key={geo.rsmKey}
                  geography={geo}
                  fill={value > 0 ? colorScale(value) : '#f0f0f0'}
                  stroke="#ffffff"
                  strokeWidth={0.5}
                  onClick={() => onCountryClick(geo.properties)}
                  style={{
                    default: { outline: 'none' },
                    hover: { outline: 'none', opacity: 0.8 },
                    pressed: { outline: 'none' }
                  }}
                />
              );
            })
          }
        </Geographies>
      </ComposableMap>
      
      {/* Legend */}
      <div className="mt-4 flex items-center justify-center space-x-4">
        <span className="text-sm">Low</span>
        <div className="flex space-x-1">
          {[...Array(10)].map((_, i) => (
            <div
              key={i}
              className="w-4 h-4"
              style={{ backgroundColor: colorScale((i + 1) * 10) }}
            />
          ))}
        </div>
        <span className="text-sm">High</span>
      </div>
    </div>
  );
}
```

### 4. Common Components

#### LoadingSpinner
```javascript
function LoadingSpinner({ size = 'medium', message = 'Loading...' }) {
  const sizeClasses = {
    small: 'w-4 h-4',
    medium: 'w-8 h-8', 
    large: 'w-12 h-12'
  };
  
  return (
    <div className="flex flex-col items-center justify-center p-8">
      <div className={`${sizeClasses[size]} animate-spin border-2 border-blue-600 border-t-transparent rounded-full`} />
      {message && <p className="mt-2 text-gray-600">{message}</p>}
    </div>
  );
}
```

#### ErrorBoundary
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary p-8 text-center">
          <h2 className="text-xl font-bold text-red-600 mb-4">Something went wrong</h2>
          <p className="text-gray-600 mb-4">
            We're sorry, but there was an error loading this section.
          </p>
          <button
            onClick={() => window.location.reload()}
            className="px-4 py-2 bg-blue-600 text-white rounded-lg"
          >
            Reload Page
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}
```

## Styling Guidelines

### Tailwind CSS Conventions

#### Layout Patterns
```javascript
// Container pattern
<div className="container mx-auto px-4 py-8 max-w-6xl">

// Card pattern  
<div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">

// Grid layouts
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

// Responsive flex
<div className="flex flex-col md:flex-row items-center justify-between gap-4">
```

#### Color System
```javascript
// Primary colors
"bg-blue-600 text-white"      // Primary buttons
"text-blue-600 border-blue-600" // Links, accents

// Status colors
"text-green-600"              // Success states
"text-red-600"                // Error states  
"text-yellow-600"             // Warning states
"text-gray-600"               // Secondary text

// Background variations
"bg-gray-50"                  // Light background
"bg-gray-100"                 // Card backgrounds
"bg-gray-800"                 // Dark mode
```

#### Interactive States
```javascript
// Button states
"bg-blue-600 hover:bg-blue-700 active:bg-blue-800 disabled:opacity-50"

// Link states
"text-blue-600 hover:text-blue-800 hover:underline"

// Focus states (accessibility)
"focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
```

### Responsive Design Patterns

```javascript
// Mobile-first responsive
"text-sm md:text-base lg:text-lg"
"p-4 md:p-6 lg:p-8"
"grid-cols-1 md:grid-cols-2 lg:grid-cols-3"

// Show/hide on different screens
"hidden md:block"             // Hide on mobile
"block md:hidden"             // Show only on mobile
```

## Performance Patterns

### Component Memoization
```javascript
import { memo } from 'react';

const SearchResult = memo(function SearchResult({ result }) {
  return (
    <div className="search-result">
      {/* Component content */}
    </div>
  );
});

// Only re-render when result prop changes
```

### Lazy Loading
```javascript
import { lazy, Suspense } from 'react';

const AnalyticsDashboard = lazy(() => import('./AnalyticsDashboard'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <AnalyticsDashboard />
    </Suspense>
  );
}
```

### Custom Hooks for Data Fetching
```javascript
function useSearchData(query, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    if (!query) return;
    
    let cancelled = false;
    setLoading(true);
    setError(null);
    
    const fetchData = async () => {
      try {
        const result = await searchArchive({ query, ...options });
        if (!cancelled) {
          setData(result);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };
    
    fetchData();
    
    return () => {
      cancelled = true;
    };
  }, [query, JSON.stringify(options)]);
  
  return { data, loading, error };
}
```

## Testing Components

### Component Testing with React Testing Library
```javascript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import SearchForm from './SearchForm';

describe('SearchForm', () => {
  const mockOnSearch = jest.fn();
  
  beforeEach(() => {
    mockOnSearch.mockClear();
  });
  
  test('renders search input and button', () => {
    render(<SearchForm onSearch={mockOnSearch} />);
    
    expect(screen.getByPlaceholderText('Enter search term...')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /search/i })).toBeInTheDocument();
  });
  
  test('calls onSearch with form data when submitted', async () => {
    render(<SearchForm onSearch={mockOnSearch} />);
    
    const input = screen.getByPlaceholderText('Enter search term...');
    const button = screen.getByRole('button', { name: /search/i });
    
    fireEvent.change(input, { target: { value: 'test query' } });
    fireEvent.click(button);
    
    await waitFor(() => {
      expect(mockOnSearch).toHaveBeenCalledWith({
        query: 'test query',
        language: 'auto'
      });
    });
  });
});
```

### Testing with Context
```javascript
import { render } from '@testing-library/react';
import { ApiContext } from '../contexts/ApiContext';

function renderWithContext(component, contextValue) {
  return render(
    <ApiContext.Provider value={contextValue}>
      {component}
    </ApiContext.Provider>
  );
}

test('component uses API context', () => {
  const mockApiValue = {
    searchArchive: jest.fn()
  };
  
  renderWithContext(<SearchResults />, mockApiValue);
  // Test component behavior with context
});
```

## Best Practices

### Component Design Principles
1. **Single Responsibility**: Each component has one clear purpose
2. **Props Interface**: Clear, documented prop types
3. **Composition over Inheritance**: Build complex UIs from simple components
4. **Accessibility**: Include ARIA labels, keyboard navigation
5. **Performance**: Use memo, lazy loading, and efficient re-renders

### Code Organization
1. Keep components under 200 lines
2. Extract custom hooks for complex logic
3. Use TypeScript or PropTypes for type safety
4. Group related components in directories
5. Maintain consistent naming conventions

### Error Handling
1. Use Error Boundaries for component errors
2. Handle loading and error states in UI
3. Provide meaningful error messages
4. Graceful degradation for failed features

---

*For more details, see [State Management Guide](./STATE-MANAGEMENT.md) and [Styling Guide](./STYLING.md)*