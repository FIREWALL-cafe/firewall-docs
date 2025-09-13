# Search API Integration

> Last Updated: 2025-01-12
> Implementation: [`firewall-client-server/client/server/fetch.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/server/fetch.js)

## Overview

Firewall Cafe integrates with multiple external APIs to provide comprehensive search comparison and translation services. The system is designed with fallback mechanisms and error handling for robust operation.

## Search Providers

### Google Images Search

**Primary Provider: Serper.dev**
- **Endpoint**: `https://google.serper.dev/images`
- **Method**: POST
- **Purpose**: Real-time Google image search results

**Request Configuration:**
- Endpoint: `https://google.serper.dev/images`
- Method: POST
- Headers: X-API-KEY, Content-Type
- Parameters: q (query), num (9 results), hl (language), gl (location)
- Implementation: See `client/server/fetch.js`

**Fallback Provider: SerpAPI**
- **Package**: `serpapi` npm package
- **Purpose**: Backup when Serper.dev fails
- **Implementation**: Automatic fallback on primary failure

**Response Processing:**
- Extracts first 9 images
- Maps to standard format with position, title, link, original, source
- Implementation in `client/server/fetch.js`

### Baidu Images Search

**Custom Scraping Implementation**
- **Endpoint**: `https://image.baidu.com/search/acjson`
- **Method**: GET
- **Purpose**: Baidu image search results via JSON API

**Request Configuration:**
- Endpoint: `https://image.baidu.com/search/acjson`
- Method: GET with multiple query parameters
- Key params: queryWord, word (search term), pn (page number), rn (results)
- Full implementation in `client/server/fetch.js`

**Response Processing:**
- Extracts first 9 images from data array
- Maps Baidu fields to standard format
- See processing logic in `client/server/fetch.js`

## Translation Services

### Custom Babelfish Service

**Primary Translation Provider**
- **Endpoint**: `https://babelfish.firewallcafe.com/`
- **Purpose**: Bidirectional English/Chinese translation
- **Features**: Language detection + translation

**Translation & Detection:**
- Translation endpoint: `https://babelfish.firewallcafe.com/`
- Detection endpoint: `https://babelfish.firewallcafe.com/detect`
- Both use POST with JSON payloads
- Implementation in `client/server/fetch.js`

**Fallback: Google Translate API**
- **Implementation**: Secondary translation service
- **Purpose**: Backup when Babelfish fails
- **Configuration**: Via Google Cloud API keys

## Geolocation Services

### IP Geolocation

**Provider: ip-api.com**
- **Endpoint**: `http://ip-api.com/json/`
- **Purpose**: Convert IP addresses to geographic data
- **Rate Limit**: 45 requests/minute (free tier)

**Implementation:**
- Endpoint: `http://ip-api.com/json/{ip}`
- Returns country, region, city, latitude, longitude
- See geolocation handling in `client/index.js`

## Search Flow Architecture

### Dual Search Process

1. **Input Processing**
   - Language detection via Babelfish
   - Auto-translation between English and Chinese
   - Implementation: `client/server/fetch.js`

2. **Parallel Search Execution**
   - Concurrent Google and Baidu searches
   - Promise.all for efficiency
   - See `client/index.js` POST /images endpoint

3. **Result Processing**
   - Standardized format for both engines
   - Translation metadata included
   - Full flow in `client/index.js`

### Error Handling

**Error Handling Patterns:**
- Graceful degradation with fallback providers
- Timeout handling (default 10 seconds)
- Error logging and recovery
- Implementation patterns in `client/server/fetch.js`

## Rate Limiting & Quotas

### API Quotas

**Serper.dev:**
- Free tier: 100 requests/month
- Paid tiers: 1,000-100,000 requests/month
- Rate limit: 10 requests/second

**ip-api.com:**
- Free tier: 45 requests/minute
- Paid tiers: Up to 1,000 requests/minute

**Babelfish Service:**
- Custom rate limiting
- Optimized for project usage patterns

### Quota Management

- Rate limit checking per service
- Low quota notifications
- Usage tracking implementation in production

## Image Processing

### CORS Handling

**Proxy Implementation:**
- Endpoint: `/proxy-image`
- Handles CORS and caching headers
- User-Agent spoofing for compatibility
- Full implementation in `client/index.js`

### Cloud Storage Integration

**Digital Ocean Spaces:**
- **Purpose**: Backup storage for images
- **Implementation**: Async upload via worker threads
- **Benefits**: Persistent storage, CDN distribution

**Digital Ocean Spaces Integration:**
- Async upload via worker threads
- Path structure: `images/{searchId}/{position}.jpg`
- Public read ACL for CDN access
- Implementation in `server/api/worker.js`

## Configuration

### Environment Variables

**Required API Keys:**
- `SERPER_API_KEY`: Serper.dev for Google search
- `SERPAPI_KEY`: Fallback SerpAPI key
- `GOOGLE_TRANSLATE_KEY`: Translation fallback

**Storage Configuration:**
- `SPACES_KEY`, `SPACES_SECRET`: Digital Ocean credentials
- `SPACES_ENDPOINT`, `SPACES_BUCKET`: Storage location

**Service URLs:**
- `BABELFISH_URL`: Translation service endpoint

See `server/api/config-example.js` for template

### Development vs Production

**Development:**
- Reduced API calls
- Mock responses for testing
- Local image storage

**Production:**
- Full API integration
- Cloud storage backup
- Comprehensive error logging

## Monitoring & Analytics

### Monitoring & Analytics

**API Performance Tracking:**
- Service call duration logging
- Success/failure metrics
- Implementation patterns in production code

**Error Reporting:**
- Comprehensive error logging
- Production error notifications
- Context preservation for debugging

See monitoring implementation in `client/index.js`

---

*For database storage of API results, see [Database Schema](./DATABASE.md)*
*For endpoint implementations, see [Endpoints Reference](./ENDPOINTS.md)*