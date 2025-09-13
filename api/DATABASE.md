# Database Schema

> Last Updated: 2025-01-12
> Database: PostgreSQL
> Schema File: `/server/api/schema.sql`

## Overview

Firewall Cafe uses PostgreSQL with a carefully designed schema optimized for:
- Geographic search analysis
- Image comparison storage
- Community voting systems
- Full-text search capabilities

## Core Tables

### `searches` Table

Primary table storing search metadata and results.

**Schema Location**: [`firewall-client-server/server/api/schema.sql`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/server/api/schema.sql)

**Key Fields:**
- `search_id`: Unique identifier (UUID)
- `search_term`: Original search query
- `search_term_translated`: Auto-translated version
- `search_timestamp`: Unix timestamp for temporal filtering
- `search_location`: Snake_case event/location identifier
- Geographic fields: `search_country`, `search_city`, `search_region`
- IP tracking: `user_ip` for analytics and geolocation

**Indexes:**
- Primary key index on `search_id`
- Geographic indexes for country/city/region filtering
- Temporal index on `search_timestamp`
- Full-text search index on `search_term`

### `images` Table

Stores image results linked to searches.

**Key Fields:**
- `search_id`: Foreign key to searches table
- `image_url`: Original external image URL
- `search_engine`: 'google' or 'baidu'
- `position`: Ranking position in results (1-9)
- `cloud_storage_path`: Digital Ocean Spaces storage path

**Relationships:**
- `searches` (1:N) `images` via `search_id`

### `votes` Table

Fixed vote categories for community assessment.

**Fixed Categories (IDs 1-7):**
1. **Censored** - Content appears censored
2. **Uncensored** - Content appears uncensored  
3. **Bad Translation** - Translation quality poor
4. **Good Translation** - Translation quality good
5. **Lost in Translation** - Meaning lost in translation
6. **Bad Result** - Search results irrelevant
7. **NSFW** - Not safe for work content

### `have_votes` Table

Junction table linking searches to votes with voter metadata.

**Key Fields:**
- `search_id`: Links to specific search
- `vote_id`: Links to vote category
- `voter_ip`: IP tracking for analytics
- `vote_comment`: Optional voter feedback

**Relationships:**
- `searches` (1:N) `have_votes` via `search_id`
- `votes` (1:N) `have_votes` via `vote_id`

## Advanced Features

### Full-Text Search

PostgreSQL extensions enabled for advanced search:
- **pg_trgm**: Trigram matching for fuzzy search
- **unaccent**: Accent-insensitive search

**Index Configuration**: See `server/api/schema.sql` for trigram indexes

**Search Capabilities:**
- Fuzzy text matching with trigram similarity
- Multi-language search support
- Accent-insensitive search

### Geographic Indexing

Optimized for geographic filtering queries:
- Country, city, region, and location indexes
- See index definitions in `server/api/schema.sql`

**Hierarchical Filtering:**
- Country-level aggregation
- US states-specific queries
- City-level granularity

### Temporal Queries

Time-based analysis with Unix timestamps:
- Timestamp index for date range queries
- Implementation in `server/api/schema.sql`

**Query Patterns:**
- Date range filtering (start_date ≤ end_date)
- Recent activity feeds
- Temporal trend analysis

## Data Access Patterns

### Search Operations

**Complex Filtering Implementation**: [`firewall-client-server/server/api/queries.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/server/api/queries.js)
- `getFilteredSearches()` function for multi-parameter filtering
- Geographic, temporal, and vote-based filtering

### Analytics Queries

**Query Implementations**: See `server/api/queries.js`
- Geographic distribution: `getGeographicAnalytics()`
- Vote analytics: `getVoteAnalytics()`
- Search trends: `getSearchAnalytics()`

## Database Migrations

Migration files located in `/server/api/migrations/`:

1. **Initial Schema** - Core tables creation
2. **Geographic Columns** - Added location fields
3. **Full-Text Indexes** - Added trigram search
4. **Image URL Updates** - Schema refinements

### Running Migrations

Migrations are executed via npm scripts in `server/api/`
- See migration files in [`firewall-client-server/server/api/migrations/`](https://github.com/FIREWALL-cafe/firewall-client-server/tree/main/server/api/migrations)

## Configuration

Database connection configured in:
- **Template**: [`firewall-client-server/server/api/config-example.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/server/api/config-example.js)
- **Local Config**: Copy to `config.js` with your credentials

## Performance Considerations

### Query Optimization

- **Pagination**: Always use LIMIT/OFFSET for large result sets
- **Indexes**: Leverage geographic and temporal indexes
- **Joins**: Efficient LEFT JOINs for vote aggregation

### Connection Management

- Connection pooling for concurrent requests
- Proper connection cleanup
- Query timeout handling

### Storage

- **Images**: External URLs with cloud storage backup
- **Text**: Efficient text storage with full-text search
- **Timestamps**: Unix timestamps for fast temporal queries

## Monitoring & Maintenance

### Key Metrics

- Search volume trends
- Vote participation rates
- Geographic distribution
- Query performance

### Regular Maintenance

- Index maintenance and statistics updates
- Connection pool monitoring
- Storage usage tracking
- Query performance analysis

---

*For API endpoint details using this schema, see [Endpoints Reference](./ENDPOINTS.md)*
*For external service integrations, see [Search API Integration](./SEARCH-APIS.md)*