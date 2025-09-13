# API Endpoints Reference

> Last Updated: 2025-01-12
> Architecture: Dual-server (Client Proxy + Backend API)

## Overview

Firewall Cafe uses a **dual-server architecture** with two separate API layers:

1. **Client Proxy Server** (Port 8080) - Express.js proxy handling external services
2. **Backend API Server** (Port 11458) - Core data operations with PostgreSQL

## Implementation Files

- **Client Proxy Server**: [`firewall-client-server/client/index.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/index.js)
- **Backend API Server**: [`firewall-client-server/server/api/server.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/server/api/server.js)
- **Query Logic**: [`firewall-client-server/server/api/queries.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/server/api/queries.js)

## Client Proxy Server Endpoints

### Search Operations

#### `POST /images`
Perform dual search comparison (Google + Baidu)
- Returns search results from both engines with translations
- Implementation: `client/index.js` lines 200-250

#### `POST /searches`
Search with advanced filtering
- Supports pagination, geographic, date, and vote filters
- Implementation: `client/index.js` lines 300-350

#### `POST /searches/:search_id/images`
Get images for specific search
- Returns all images associated with a search ID
- Implementation: `client/index.js`

### Analytics Endpoints

#### `GET /dashboardData`
Get dashboard statistics and recent activity
- Returns total counts and recent activity feed
- Implementation: `client/index.js`

#### `GET /api/analytics/geographic`
Geographic distribution analytics
- Query Parameters: `page`, `page_size`
- Implementation: `client/index.js`

#### `GET /api/analytics/geographic/us-states`
US states-specific analytics

#### `GET /api/analytics/searches`
Search volume and trend analytics

#### `GET /api/analytics/votes`
Vote pattern analytics

#### `GET /api/analytics/recent-activity`
Recent user activity feed

### Utility Endpoints

#### `GET /proxy-image`
Proxy external images to handle CORS
- Query Parameter: `url` (external image URL)
- Implementation: `client/index.js`

#### `GET /api/my-ip`
Get user's IP address and location
- Returns IP, country, and city information
- Implementation: `client/index.js`

#### `GET /api/countries`
Get list of available countries for filtering

#### `GET /searches/search-locations`
Get list of available search locations

### Voting System

#### `POST /vote`
Submit community vote on search result
- Request includes search_id, vote_id, voter_ip, optional comment
- Implementation: `client/index.js`

#### `POST /searches/votes/counts/:search_id`
Get vote counts for specific search

### Communication

#### `POST /send-email`
Send email via Postmark integration
- Request includes to, subject, message fields
- Implementation: `client/index.js`

#### `POST /api/search-demo`
Demo search comparison endpoint

## Backend API Server Endpoints

### Core CRUD Operations

The backend API (`/server/api/server.js`) provides comprehensive data operations:

#### GET Endpoints (37 total)
- **Data Retrieval**: Searches, images, votes with advanced filtering
- **Geographic Queries**: Country, region, city-based filtering  
- **Temporal Queries**: Date range and timestamp filtering
- **Analytics Queries**: Aggregated statistics and trends

#### POST Endpoints (7 total)
- **Data Creation**: New searches, images, votes
- **Batch Operations**: Bulk data insertion
- **Complex Queries**: Multi-parameter filtering

### Key Features

#### Advanced Filtering
- **Geographic**: Countries, US states, search locations
- **Temporal**: Date ranges with proper validation
- **Content**: Vote categories, search terms
- **Pagination**: Page-based result sets

#### Search Capabilities
- **Full-text Search**: PostgreSQL trgm extension
- **Fuzzy Matching**: Trigram similarity search
- **Multi-language**: English/Chinese search support

## Authentication & Security

### Rate Limiting
- API rate limiting implemented on proxy server
- IP-based tracking and throttling

### CORS Handling
- Image proxy for external content
- Proper CORS headers for frontend integration

### Data Validation
- Request body validation
- Parameter sanitization
- SQL injection prevention

## Error Handling

### Standard HTTP Status Codes
- `200`: Success
- `400`: Bad Request (invalid parameters)
- `404`: Not Found
- `500`: Internal Server Error

### Error Response Format
- Returns error message and optional details
- See error handling in `client/index.js` and `server/api/server.js`

## Development Notes

### Local Development
- Client Proxy: `http://localhost:8080`
- Backend API: `http://localhost:11458`

### Production
- All requests route through client proxy
- Backend API is internal-only

### Debugging
- Check network requests in browser dev tools
- Monitor server logs for API errors
- Use `/api/my-ip` to verify client detection

---

*For database schema details, see [Database Schema](./DATABASE.md)*
*For external API integrations, see [Search API Integration](./SEARCH-APIS.md)*