# API Documentation

The Firewall Cafe API provides access to search data, analytics, and voting functionality. This document covers all available endpoints, authentication, and usage examples.

## Base URL

- **Production**: `https://api.firewallcafe.com`
- **Development**: `http://localhost:11458`

## Table of Contents
- [Authentication](#authentication)
- [Response Formats](#response-formats)
- [Rate Limiting](#rate-limiting)
- [Dashboard Endpoints](#dashboard-endpoints)
- [Analytics Endpoints](#analytics-endpoints)
- [Search Endpoints](#search-endpoints)
- [Image Endpoints](#image-endpoints)
- [Vote Endpoints](#vote-endpoints)
- [Error Handling](#error-handling)
- [Examples](#examples)

## Authentication

### Public Endpoints (No Auth Required)
Most GET endpoints are publicly accessible and require no authentication.

### Protected Endpoints (Shared Secret)
Write operations (POST, PUT, DELETE) require a shared secret in the request body:

```javascript
{
  "sharedSecret": "your_secret_here",
  // ... other data
}
```

### Example Protected Request
```bash
curl -X POST https://api.firewallcafe.com/vote \
  -H "Content-Type: application/json" \
  -d '{
    "sharedSecret": "your_secret",
    "searchId": 123,
    "voteType": "censored"
  }'
```

## Response Formats

All responses are in JSON format with consistent structure:

### Success Response
```json
{
  "success": true,
  "data": [...],
  "count": 42,
  "message": "Success"
}
```

### Error Response
```json
{
  "success": false,
  "error": "Error description",
  "code": 400
}
```

## Rate Limiting

- **Public endpoints**: 100 requests per minute per IP
- **Protected endpoints**: 50 requests per minute per secret
- Rate limit headers included in responses:
  - `X-RateLimit-Limit`: Request limit
  - `X-RateLimit-Remaining`: Remaining requests
  - `X-RateLimit-Reset`: Reset time (Unix timestamp)

## Dashboard Endpoints

### Get Dashboard Overview
Returns overview statistics for the dashboard.

```http
GET /dashboard
```

**Response:**
```json
{
  "totalSearches": 5679,
  "totalImages": 45234,
  "totalVotes": 1234,
  "totalUsers": 567,
  "recentSearches": 89,
  "topCountries": [
    {"country": "United States", "count": 1234},
    {"country": "China", "count": 987}
  ]
}
```

### Get Total Searches
Returns the total number of searches in the database.

```http
GET /searches/total
```

## Analytics Endpoints

### Geographic Analytics
Get search distribution by country and region.

```http
GET /analytics/geographic
```

**Query Parameters:**
- `timeframe`: `7d`, `30d`, `90d`, `1y` (default: `30d`)
- `minCount`: Minimum searches to include (default: 1)

**Response:**
```json
{
  "countries": [
    {
      "country": "United States",
      "countryCode": "US", 
      "searchCount": 1234,
      "cities": [
        {"city": "New York", "count": 456},
        {"city": "Los Angeles", "count": 234}
      ]
    }
  ],
  "totalSearches": 5679
}
```

### US States Analytics
Get detailed analytics for US states (when available).

```http
GET /analytics/geographic/us-states
```

### Search Analytics
Get search trends, popular terms, and language distribution.

```http
GET /analytics/searches
```

**Response:**
```json
{
  "timeline": [
    {"date": "2025-08-01", "count": 45},
    {"date": "2025-08-02", "count": 67}
  ],
  "topTerms": [
    {"term": "climate change", "count": 234, "languages": ["en", "zh"]},
    {"term": "democracy", "count": 187, "languages": ["en"]}
  ],
  "languageDistribution": [
    {"language": "en", "percentage": 65.4},
    {"language": "zh", "percentage": 34.6}
  ]
}
```

### Vote Analytics
Get voting patterns and censorship trends.

```http
GET /analytics/votes
```

**Response:**
```json
{
  "voteCategories": [
    {"category": "censored", "count": 456, "percentage": 23.4},
    {"category": "uncensored", "count": 789, "percentage": 40.5}
  ],
  "timeline": [
    {"date": "2025-08-01", "censored": 12, "uncensored": 18}
  ],
  "topVotedSearches": [
    {"searchId": 123, "term": "tiananmen", "votes": 45}
  ]
}
```

### Recent Activity
Get the most recent search activity.

```http
GET /analytics/recent-activity
```

**Query Parameters:**
- `limit`: Number of results (default: 20, max: 100)

## Search Endpoints

### Get All Searches
Retrieve all searches with optional filtering.

```http
GET /searches
```

**Query Parameters:**
- `limit`: Number of results (default: 50, max: 1000)
- `offset`: Pagination offset (default: 0)
- `location`: Filter by location
- `term`: Filter by search term
- `language`: Filter by language code (`en`, `zh`, etc.)
- `dateFrom`: Start date (YYYY-MM-DD)
- `dateTo`: End date (YYYY-MM-DD)

**Response:**
```json
{
  "searches": [
    {
      "searchId": 123,
      "timestamp": 1693123456789,
      "location": "New York",
      "searchTermInitial": "climate change",
      "searchTermTranslation": "气候变化",
      "searchEngineInitial": "google",
      "languageInitial": "en",
      "languageTranslation": "zh"
    }
  ],
  "total": 5679,
  "limit": 50,
  "offset": 0
}
```

### Get Search by ID
Get a specific search by its ID.

```http
GET /searches/search_id/{search_id}
```

**Response:**
```json
{
  "search": {
    "searchId": 123,
    "timestamp": 1693123456789,
    "location": "New York",
    "ipAddress": "192.168.1.1",
    "searchTermInitial": "climate change",
    "searchTermTranslation": "气候变化",
    "languageInitial": "en",
    "languageConfidence": 0.98
  }
}
```

### Get Searches with Images
Get searches including their associated image data.

```http
GET /searches/images
```

**Query Parameters:** Same as `/searches`

### Get Filtered Searches
Advanced search filtering endpoint.

```http
GET /searches/filter
```

**Query Parameters:**
- `countries[]`: Array of country names
- `cities[]`: Array of city names  
- `voteTypes[]`: Array of vote types (`censored`, `uncensored`, etc.)
- `years[]`: Array of years
- `terms[]`: Array of search terms
- `page`: Page number (default: 1)
- `pageSize`: Results per page (default: 10, max: 100)

### Create Search
Create a new search record (requires authentication).

```http
POST /createSearch
```

**Request Body:**
```json
{
  "sharedSecret": "your_secret",
  "searchData": {
    "timestamp": 1693123456789,
    "location": "New York",
    "ipAddress": "192.168.1.1",
    "searchTermInitial": "climate change",
    "searchTermTranslation": "气候变化",
    "searchEngineInitial": "google",
    "languageInitial": "en",
    "languageConfidence": 0.98
  }
}
```

## Image Endpoints

### Get All Images
Retrieve image records.

```http
GET /images
```

**Query Parameters:**
- `limit`: Number of results (default: 50)
- `searchId`: Filter by search ID
- `engine`: Filter by search engine (`google`, `baidu`)

### Get Image by ID
Get a specific image record.

```http
GET /images/image_id/{image_id}
```

### Get Images by Search ID
Get all images associated with a specific search.

```http
GET /images/search_id/{search_id}
```

### Get Images by Vote Type
Get images filtered by vote category.

```http
GET /images/type/{vote_type}
```

**Vote Types:**
- `censored_searches`
- `uncensored_searches`
- `bad_translation_searches`
- `good_translation_searches`
- `lost_in_translation_searches`
- `nsfw_searches`
- `wtf_searches`

### Save Images
Save image data for a search (requires authentication).

```http
POST /saveImages
```

**Request Body:**
```json
{
  "sharedSecret": "your_secret",
  "searchId": 123,
  "images": [
    {
      "href": "https://example.com/image1.jpg",
      "hrefOriginal": "https://original.com/image1.jpg",
      "rank": "1",
      "engine": "google",
      "mimeType": "image/jpeg"
    }
  ]
}
```

## Vote Endpoints

### Get All Votes
Retrieve voting data.

```http
GET /searches/votes
```

### Get Votes by Search ID
Get votes for a specific search.

```http
GET /searches/votes/search_id/{search_id}
```

### Get Vote Counts by Search ID
Get aggregated vote counts for a search.

```http
GET /searches/votes/counts/{search_id}
```

**Response:**
```json
{
  "searchId": 123,
  "voteCounts": {
    "censored": 5,
    "uncensored": 12,
    "badTranslation": 2,
    "goodTranslation": 8,
    "nsfw": 1,
    "wtf": 3
  },
  "totalVotes": 31
}
```

### Submit Vote
Submit a vote for a search result (requires authentication).

```http
POST /vote
```

**Request Body:**
```json
{
  "sharedSecret": "your_secret",
  "searchId": 123,
  "voteType": "censored",
  "userIdentifier": "anonymous_user_123"
}
```

**Vote Types:**
- `censored`: Search results appear censored
- `uncensored`: Search results not censored  
- `bad_translation`: Translation appears incorrect
- `good_translation`: Translation matches original
- `lost_in_translation`: Term doesn't translate well
- `nsfw`: Not Safe for Work content
- `wtf`: Weird or unusual results

## Error Handling

### HTTP Status Codes
- `200`: Success
- `400`: Bad Request (invalid parameters)
- `401`: Unauthorized (missing or invalid credentials)
- `404`: Not Found (resource doesn't exist)
- `429`: Too Many Requests (rate limited)
- `500`: Internal Server Error

### Error Response Format
```json
{
  "success": false,
  "error": "Detailed error message",
  "code": 400,
  "field": "fieldName" // when applicable
}
```

## Examples

### JavaScript/Fetch Examples

#### Get Recent Searches
```javascript
async function getRecentSearches() {
  const response = await fetch('https://api.firewallcafe.com/searches?limit=10');
  const data = await response.json();
  return data.searches;
}
```

#### Submit a Vote
```javascript
async function submitVote(searchId, voteType) {
  const response = await fetch('https://api.firewallcafe.com/vote', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      sharedSecret: process.env.API_SECRET,
      searchId: searchId,
      voteType: voteType,
      userIdentifier: 'user_' + Date.now()
    })
  });
  
  return response.json();
}
```

#### Get Geographic Analytics
```javascript
async function getGeographicData(timeframe = '30d') {
  const response = await fetch(
    `https://api.firewallcafe.com/analytics/geographic?timeframe=${timeframe}`
  );
  return response.json();
}
```

### cURL Examples

#### Get Dashboard Data
```bash
curl https://api.firewallcafe.com/dashboard
```

#### Filter Searches by Country
```bash
curl "https://api.firewallcafe.com/searches/filter?countries[]=United%20States&countries[]=China&pageSize=20"
```

#### Submit Vote with Authentication
```bash
curl -X POST https://api.firewallcafe.com/vote \
  -H "Content-Type: application/json" \
  -d '{
    "sharedSecret": "your_secret",
    "searchId": 123,
    "voteType": "censored"
  }'
```

## Database Schema Reference

For detailed database schema information, see [Database Documentation](./DATABASE.md).

## Development and Testing

### Local Development
```bash
# Start API server
cd server/api
node server.js

# API available at http://localhost:11458
```

### Testing Endpoints
```bash
# Test dashboard endpoint
curl http://localhost:11458/dashboard

# Test search endpoint
curl "http://localhost:11458/searches?limit=5"
```

## Support

For API support:
- Check the [Architecture Documentation](../ARCHITECTURE.md)
- Review [Common Issues](../SETUP.md#common-issues)
- Create an issue on [GitHub](https://github.com/FIREWALL-cafe/firewall-cafe/issues)

---

*Last updated: 2025-08-26*