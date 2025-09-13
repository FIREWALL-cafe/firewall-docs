# Geographic Heatmaps Feature

> Last Updated: 2025-01-12
> Primary Component: [`firewall-client-server/client/src/components/Dashboard.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/Dashboard.jsx)

## Overview

Geographic heatmaps visualize search patterns and censorship trends across different countries and regions, providing insights into how internet restrictions vary geographically.

## Implementation

### Frontend Components
- **World Map**: Uses React Simple Maps for interactive country visualization
- **US States Map**: Detailed state-level analysis for United States
- **Color Scaling**: D3-Scale for data-driven color mapping
- **Interactive Elements**: Hover states and click handlers for detailed views

### Data Sources
- **Geographic Analytics**: [`client/index.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/index.js) - GET /api/analytics/geographic
- **US States Data**: GET /api/analytics/geographic/us-states
- **Search Distribution**: Aggregated from PostgreSQL searches table

## Map Types

### World Countries Heatmap
- **Data**: Search volume per country
- **Visualization**: Color intensity based on activity
- **Interactions**: Hover for country details, click for filtered results

### US States Heatmap  
- **Data**: State-level search distribution within United States
- **Granularity**: Individual state boundaries
- **Features**: State abbreviations, population-adjusted metrics

### City-Level Analysis
- **Data**: Top cities by search volume
- **Display**: Bubble map or list view
- **Filtering**: Drill-down from country to city level

## Technical Implementation

### Libraries Used
- **react-simple-maps**: SVG-based world map rendering
- **d3-scale**: Color scaling and data normalization  
- **topojson-client**: Geographic data processing
- **Custom GeoJSON**: Country and state boundary data

### Data Processing
- **Aggregation**: Search counts grouped by geographic fields
- **Normalization**: Color scales adapted to data ranges
- **Caching**: Geographic data cached for performance

## Visual Design

### Color Schemes
- **Low Activity**: Light blue (#f0f9ff)
- **Medium Activity**: Medium blue (#3b82f6)
- **High Activity**: Dark blue (#1e3a8a)
- **No Data**: Light gray (#f3f4f6)

### Interactive Elements
- **Hover Effects**: Country highlighting and tooltips
- **Click Actions**: Filter searches by selected geography
- **Zoom Controls**: Pan and zoom for detailed exploration
- **Legend**: Color scale explanation

## Data Metrics

### Search Volume
- Total searches per geographic region
- Unique users per location
- Search frequency over time periods

### Censorship Patterns
- Vote distribution by geography
- Censorship indicator prevalence
- Regional content filtering trends

### Temporal Analysis
- Geographic patterns over time
- Seasonal variations by region
- Event-driven search spikes

## User Interface

### Dashboard Integration
- **Tab Navigation**: Geographic analysis as dashboard tab
- **Filter Controls**: Date range and metric selection
- **Export Options**: PNG/SVG download of maps

### Mobile Optimization
- **Touch Interactions**: Tap and pinch gestures
- **Responsive Layout**: Adapted map sizing
- **Simplified UI**: Fewer hover states, more tap actions

## Performance Considerations

### Data Loading
- **Lazy Loading**: Maps load on tab activation
- **Progressive Enhancement**: Base data first, details on demand
- **Caching Strategy**: Geographic boundaries cached locally

### Rendering Optimization
- **SVG Optimization**: Simplified boundary geometries
- **Color Computation**: Pre-computed color scales
- **Smooth Animations**: CSS transitions for state changes

## Future Enhancements

### Advanced Visualizations
- **3D Globe**: Three-dimensional geographic representation
- **Time Animation**: Animated progression of patterns
- **Layer Overlays**: Multiple data dimensions simultaneously

### Enhanced Interactivity
- **Selection Tools**: Multi-country selection
- **Comparison Mode**: Side-by-side regional analysis
- **Deep Linking**: Shareable URLs with geographic filters

### Data Enrichment
- **Population Data**: Per-capita search metrics
- **Economic Indicators**: GDP correlation with search patterns
- **Freedom Indices**: Integration with press freedom scores

---

*For dashboard implementation, see [Analytics Dashboard](./ANALYTICS.md)*
*For geographic data structure, see [Database Schema](../api/DATABASE.md)*