# Search Comparison Feature

> Last Updated: 2025-01-12
> Primary Component: [`firewall-client-server/client/src/components/Search.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/Search.jsx)

## Overview

The search comparison feature is the core functionality of Firewall Cafe, providing side-by-side comparison of Google and Baidu image search results to reveal censorship patterns.

## Implementation

### Frontend Components
- **Search Interface**: [`client/src/components/SearchInput.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/SearchInput.jsx)
- **Results Display**: [`client/src/components/SearchResults.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/SearchResults.jsx)
- **Image Grid**: [`client/src/components/ImageGrid.jsx`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/src/components/ImageGrid.jsx)

### Backend Processing
- **Search Endpoint**: [`client/index.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/index.js) - POST /images
- **API Integration**: [`client/server/fetch.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/server/fetch.js)

## Search Flow

### 1. Query Processing
- User enters search term
- Language detection via Babelfish service
- Automatic translation between English and Chinese

### 2. Dual Search Execution
- Parallel API calls to Google (Serper.dev) and Baidu
- 9 images retrieved from each engine
- Results normalized to standard format

### 3. Result Storage
- Search metadata saved to PostgreSQL
- Images queued for cloud storage backup
- Geographic data enriched via IP geolocation

### 4. Display & Interaction
- Side-by-side grid layout (Google left, Baidu right)
- Image proxy for CORS handling
- Voting interface for community assessment

## Key Features

### Language Handling
- **Auto-detection**: Identifies input language
- **Bidirectional Translation**: English ↔ Chinese
- **Display**: Shows original and translated terms

### Visual Comparison
- **Grid Layout**: 3x3 image grid per search engine
- **Position Mapping**: Maintains result ranking
- **Image Fallbacks**: Placeholder for failed loads

### Metadata Display
- Search timestamp
- Geographic location
- Translation quality
- Vote counts per category

## Technical Details

### API Providers
- **Google Images**: Serper.dev (primary), SerpAPI (fallback)
- **Baidu Images**: Direct JSON API scraping
- **Translation**: Babelfish service (primary), Google Translate (fallback)

### Performance Optimizations
- Parallel search execution
- Image lazy loading
- Result caching (15-minute TTL)
- CDN distribution for stored images

### Error Handling
- Graceful API fallbacks
- Timeout protection (10 seconds)
- User-friendly error messages
- Retry mechanisms

## User Interface

### Search Input
- Large, centered search box
- Auto-focus on page load
- Search button with loading state
- Language indicator

### Results Layout
- **Header**: Search term, translation, timestamp
- **Body**: Two-column image grid
- **Footer**: Voting buttons, share options

### Mobile Responsiveness
- Stacked layout on small screens
- Touch-optimized interactions
- Swipeable image galleries

## Data Collection

### Search Metrics
- Query terms and translations
- Timestamp and duration
- Result counts and positions
- User location (country/city)

### Image Data
- Source URLs
- Titles and alt text
- Search engine origin
- Cloud storage paths

### User Feedback
- Vote categories (7 types)
- Vote counts per search
- Optional comments
- Voter geographic data

## Future Enhancements

### Planned Features
- Video search comparison
- News search comparison
- Real-time censorship alerts
- Advanced filtering options

### Technical Improvements
- WebSocket for live updates
- Machine learning for pattern detection
- Enhanced image analysis
- Multi-language support beyond English/Chinese

---

*For voting system details, see [Voting System](./VOTING.md)*
*For technical implementation, see [API Endpoints](../api/ENDPOINTS.md)*