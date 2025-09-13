# Analytics Dashboard Feature

> Last Updated: 2025-01-12
> Primary Component: [`firewall-client-server/client/src/components/Dashboard.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/Dashboard.jsx)

## Overview

The analytics dashboard provides comprehensive insights into search patterns, user behavior, and censorship trends through interactive charts and data visualizations.

## Implementation

### Frontend Components
- **Dashboard Layout**: Tab-based navigation for different analytics views
- **Chart Components**: Chart.js integration for data visualization
- **Geographic Maps**: React Simple Maps for heatmap displays
- **Filter Controls**: Date range and parameter selection

### Data Sources
- **Dashboard Data**: [`client/index.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/index.js) - GET /dashboardData
- **Geographic Analytics**: GET /api/analytics/geographic
- **Vote Analytics**: GET /api/analytics/votes  
- **Search Analytics**: GET /api/analytics/searches

## Dashboard Sections

### Overview Tab
- **Key Metrics**: Total searches, images, votes, users
- **Recent Activity**: Latest search and vote activity
- **Trend Indicators**: Growth percentages and changes
- **Quick Links**: Navigation to detailed views

### Geographic Tab
- **World Heatmap**: Country-level search distribution
- **US States Map**: Detailed United States analysis
- **Top Countries**: Ranked list of most active regions
- **Regional Trends**: Geographic patterns over time

### Vote Analytics Tab
- **Category Distribution**: Pie chart of vote types
- **Vote Trends**: Timeline of voting activity
- **Geographic Voting**: How votes vary by location
- **Quality Metrics**: Translation and result quality assessment

### Search Trends Tab
- **Volume Over Time**: Search frequency trends
- **Popular Terms**: Most searched keywords
- **Language Patterns**: English vs Chinese usage
- **Temporal Analysis**: Peak usage periods

## Visualization Libraries

### Chart.js Integration
- **Line Charts**: Temporal trend visualization
- **Bar Charts**: Categorical data comparison
- **Pie Charts**: Proportional data display
- **Scatter Plots**: Correlation analysis

### React Simple Maps
- **Choropleth Maps**: Color-coded geographic data
- **Interactive Elements**: Hover and click handlers
- **Custom Projections**: Optimized map views
- **Data Overlays**: Multiple data layers

## Data Processing

### Real-time Updates
- **Refresh Intervals**: Automatic data fetching
- **WebSocket Integration**: Live updates for active sessions
- **Cache Management**: Efficient data storage and retrieval

### Aggregation Logic
- **Time Periods**: Daily, weekly, monthly aggregations
- **Geographic Grouping**: Country, region, city levels
- **Category Filtering**: Vote type, search engine, language

### Performance Optimization
- **Data Pagination**: Large datasets split into pages
- **Lazy Loading**: Charts load on tab activation
- **Memoization**: Cached computations for repeated views

## User Interface

### Navigation
- **Tab System**: Four main analytics sections
- **Breadcrumbs**: Clear navigation hierarchy
- **Filter Panels**: Collapsible parameter controls
- **Export Options**: Download charts and data

### Interactive Elements
- **Date Range Picker**: Custom time period selection
- **Geographic Filters**: Country and region selection
- **Drill-down Views**: Click for detailed analysis
- **Responsive Design**: Mobile-optimized layouts

### Accessibility
- **Keyboard Navigation**: Tab-accessible controls
- **Screen Reader Support**: ARIA labels and descriptions
- **Color Accessibility**: High contrast color schemes
- **Alternative Text**: Chart descriptions for non-visual users

## Key Metrics

### Search Analytics
- **Total Searches**: Cumulative search count
- **Daily Volume**: Searches per day trend
- **Geographic Distribution**: Searches by country/region
- **Language Usage**: English vs Chinese distribution

### User Engagement
- **Active Users**: Unique visitors per period
- **Session Duration**: Average time on site
- **Page Views**: Popular sections and pages
- **Return Visitors**: User retention metrics

### Content Quality
- **Vote Participation**: Percentage of searches voted on
- **Quality Scores**: Aggregated quality assessments
- **Censorship Indicators**: Censored vs uncensored ratios
- **Translation Accuracy**: Community translation ratings

## Technical Architecture

### Data Pipeline
- **Collection**: Search and vote data from PostgreSQL
- **Processing**: Aggregation queries in backend API
- **Caching**: Redis for frequently accessed analytics
- **Delivery**: REST API endpoints for frontend consumption

### Chart Configuration
- **Responsive Settings**: Adaptive chart sizing
- **Color Schemes**: Consistent branding colors
- **Animation Options**: Smooth transitions and updates
- **Interaction Handlers**: Click and hover behaviors

## Future Enhancements

### Advanced Analytics
- **Machine Learning**: Pattern detection and predictions
- **Cohort Analysis**: User behavior over time
- **A/B Testing**: Feature performance comparison
- **Anomaly Detection**: Unusual pattern identification

### Enhanced Visualizations
- **3D Charts**: Three-dimensional data representation
- **Animation Timelines**: Temporal data animation
- **Custom Widgets**: Configurable dashboard components
- **Real-time Streaming**: Live data visualization

### Export and Sharing
- **PDF Reports**: Automated report generation
- **Embeddable Widgets**: External site integration
- **API Access**: Public analytics API for researchers
- **Data Downloads**: CSV/JSON export options

---

*For geographic visualization details, see [Geographic Heatmaps](./HEATMAPS.md)*
*For voting data context, see [Voting System](./VOTING.md)*