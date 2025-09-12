# System Architecture

## Overview

Firewall Cafe is built as a modern web application with a React frontend, Express.js backend, and PostgreSQL database. The system compares search results from Google and Baidu in real-time, stores the data for analysis, and provides geographic visualization of censorship patterns.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                           Users                                 │
└─────────────────┬───────────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Frontend (React App)                         │
│  ┌────────────┐  ┌──────────┐  ┌────────────┐  ┌────────────┐ │
│  │   Search   │  │ Analytics│  │  Heatmaps  │  │   Voting   │ │
│  │ Comparison │  │Dashboard │  │    View    │  │   System   │ │
│  └────────────┘  └──────────┘  └────────────┘  └────────────┘ │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              React Context (ApiContext)                 │  │
│  └─────────────────────────────────────────────────────────┘  │
└─────────────────┬───────────────────────────────────────────────┘
                  │ HTTP/HTTPS
                  ▼
┌─────────────────────────────────────────────────────────────────┐
│              Express Proxy Server (Port 8080)                   │
│  ┌────────────┐  ┌──────────┐  ┌────────────┐  ┌────────────┐ │
│  │   Route    │  │   CORS   │  │    Auth    │  │   Cache    │ │
│  │  Handler   │  │ Handling │  │Middleware  │  │   Layer    │ │
│  └────────────┘  └──────────┘  └────────────┘  └────────────┘ │
└─────────────────┬───────────────────────────────────────────────┘
                  │
        ┌─────────┴─────────┬──────────────┬────────────┐
        ▼                   ▼              ▼            ▼
┌───────────────┐  ┌───────────────┐  ┌─────────┐  ┌──────────┐
│   Serper.dev  │  │  Baidu Search │  │ Google  │  │   IP     │
│  (Google API) │  │      API      │  │Translate│  │Geolocation│
└───────────────┘  └───────────────┘  └─────────┘  └──────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────┐
│              Backend API Service (Port 11458)                   │
│  ┌────────────┐  ┌──────────┐  ┌────────────┐  ┌────────────┐ │
│  │  queries.js│  │Analytics │  │   Voting   │  │   Image    │ │
│  │  Database  │  │Endpoints │  │  Handler   │  │  Storage   │ │
│  └────────────┘  └──────────┘  └────────────┘  └────────────┘ │
└─────────────────┬───────────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Database                          │
│  ┌────────────┐  ┌──────────┐  ┌────────────┐  ┌────────────┐ │
│  │  searches  │  │  images  │  │have_votes  │  │   votes    │ │
│  └────────────┘  └──────────┘  └────────────┘  └────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Component Details

### Frontend Layer (React Application)

**Technology**: React 18.2.0, Tailwind CSS 3.4.3

**Key Components**:
- **SearchForm**: Handles user search input and language detection
- **SearchComparison**: Displays side-by-side Google vs Baidu results
- **GeographicHeatmap**: Interactive world map with search density
- **AnalyticsDashboard**: Charts and metrics visualization
- **VotingInterface**: Community assessment of censorship

**State Management**: React Context API (ApiContext) for global state

**Routing**: React Router DOM v6 for client-side navigation

### Express Proxy Server

**Port**: 8080 (development), configurable for production

**Responsibilities**:
- Route proxying between frontend and backend services
- CORS handling for cross-origin requests
- Request/response transformation
- Static file serving
- Session management

**Key Files**:
- `/client/index.js` - Main server file
- `/client/server/fetch.js` - API integration functions
- `/client/server/config.js` - Server configuration

### External API Integrations

#### Search APIs
- **Serper.dev**: Google search results (cost-effective alternative to SerpAPI)
- **Baidu Custom Integration**: Direct Baidu search API access
- **Google Translate API**: Automatic translation between English/Chinese

#### Geolocation
- **IP Geolocation Service**: Maps IP addresses to geographic locations
- **Data Points**: Country, region, city, coordinates

### Backend API Service

**Port**: 11458 (production at api.firewallcafe.com)

**Database**: PostgreSQL with connection pooling

**Key Endpoints**:
```
GET  /dashboard              - Overview statistics
GET  /analytics/searches     - Search trends and patterns
GET  /analytics/votes        - Voting analytics
GET  /analytics/geographic   - Geographic distribution
GET  /searches               - Search records
POST /searches               - Save new search
GET  /images                 - Image records
POST /votes                  - Submit vote
```

**Authentication**: Shared secret for write operations

### Database Schema

**Core Tables**:

1. **searches**
   - Primary storage for search queries
   - Fields: id, query, translation, timestamp, location, metadata
   - Indexes on timestamp, location, query

2. **images**
   - Search result images
   - Fields: id, search_id, url, rank, source_engine
   - AWS CDN via DigitalOcean

3. **have_votes**
   - Junction table linking searches to votes
   - Fields: search_id, vote_id, user_identifier

4. **votes**
   - Vote categories definition
   - Categories: censored, uncensored, bad_translation, etc.

## Data Flow

### Search Flow
1. User enters search term in frontend
2. Frontend sends request to Express proxy
3. Proxy forwards to search APIs (Google/Baidu)
4. Translation service converts query if needed
5. Results returned and displayed side-by-side
6. Data saved to PostgreSQL for analysis

### Analytics Flow
1. Frontend requests analytics data
2. Express proxy queries backend API
3. Backend aggregates data from PostgreSQL
4. Formatted data returned to frontend
5. Charts and visualizations rendered

### Vote Flow
1. User submits vote on search result
2. Vote sent through Express proxy
3. Backend validates and stores in database
4. Analytics updated with new vote data

## Deployment Architecture

### Production Environment (Google Cloud Platform)

```
┌─────────────────────────────────────────────────────┐
│                 Google Cloud Platform                │
│                                                      │
│  ┌──────────────────────────────────────────────┐  │
│  │            App Engine (Frontend)              │  │
│  │         firewallcafe.com (port 443)          │  │
│  └──────────────────────────────────────────────┘  │
│                                                      │
│  ┌──────────────────────────────────────────────┐  │
│  │         Cloud SQL (PostgreSQL)                │  │
│  │          Production Database                  │  │
│  └──────────────────────────────────────────────┘  │
│                                                      │
│  ┌──────────────────────────────────────────────┐  │
│  │       Cloud Storage (Future: Images)          │  │
│  │           CDN for image data                  │  │
│  └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Infrastructure Components

**App Engine**:
- Auto-scaling Node.js environment
- HTTPS termination
- Load balancing

**Cloud SQL**:
- Managed PostgreSQL instance
- Automated backups
- High availability configuration

**Cloud Storage**:
- Image file storage
- CDN distribution via DigitalOcean
- Reduce database size

## Security Considerations

### API Security
- Shared secret authentication for write operations
- Rate limiting on public endpoints
- CORS configuration for allowed origins

### Data Protection
- PostgreSQL connection over SSL
- Environment variables for secrets
- No sensitive data in client code

### User Privacy
- Anonymous voting system
- IP addresses hashed for privacy
- No personal data collection

## Performance Optimizations

### Frontend
- Code splitting and lazy loading
- Image lazy loading
- React.memo for expensive components
- Debounced search inputs

### Backend
- Database connection pooling
- Query result caching
- Indexed database columns
- Batch operations where possible

### Caching Strategy
- Browser caching for static assets
- API response caching (5 minutes)
- Database query result caching
- CDN caching for images

## Monitoring and Observability

### Application Monitoring
- Error tracking and logging
- Performance metrics
- API response times
- Database query performance

### Infrastructure Monitoring
- Server health checks
- Database connection pool metrics
- API rate limit tracking
- Storage usage monitoring

## Scalability Considerations

### Current Limitations
- Single backend API instance
- No horizontal scaling for database

## Development vs Production

### Development Environment
- Frontend: http://localhost:3000
- Express Proxy: http://localhost:8080
- Backend API: Direct database connection
- Hot module replacement enabled
- Debug logging active

### Production Environment
- Frontend: https://firewallcafe.com
- API: https://api.firewallcafe.com
- Cloud SQL managed database
- Production logging to Cloud Logging
- Error reporting to monitoring service


## Migration History

### Evolution Path
1. **2016-2020**: WordPress on DreamHost
2. **2020**: PostgreSQL and API on Digital Ocean
3. **2021-Present**: Google Cloud Platform

### Data Schema Evolution
- Schema 0: WordPress posts (2016)
- Schema 1: WordPress flattened data
- Schema 2: WordPress custom posts
- Schema 3: PostgreSQL current (2020+)

See [Database Migration Guide](./migrations/database-evolution.md) for details.