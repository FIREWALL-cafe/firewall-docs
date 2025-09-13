# Voting System Feature

> Last Updated: 2025-01-12
> Database Schema: [`firewall-client-server/server/api/schema.sql`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/server/api/schema.sql)

## Overview

The voting system enables community-driven assessment of search results, allowing users to categorize and evaluate the quality and censorship patterns they observe.

## Vote Categories

### Fixed Category System (IDs 1-7)
1. **Censored** - Content appears censored or filtered
2. **Uncensored** - Content appears unfiltered  
3. **Bad Translation** - Translation quality is poor
4. **Good Translation** - Translation quality is good
5. **Lost in Translation** - Meaning lost during translation
6. **Bad Result** - Search results are irrelevant
7. **NSFW** - Not safe for work content

## Implementation

### Database Structure
- **votes Table**: Fixed categories with descriptions
- **have_votes Table**: Junction table linking searches to votes
- **Voter Tracking**: IP-based voting with optional comments

### Frontend Components
- **Voting Buttons**: Interactive vote selection interface
- **Vote Counts**: Real-time display of community votes
- **Vote Analytics**: Aggregated voting patterns

### API Endpoints
- **Submit Vote**: [`client/index.js`](https://github.com/FIREWALL-cafe/firewall-client-server/blob/main/client/index.js) - POST /vote
- **Get Counts**: POST /searches/votes/counts/:search_id
- **Vote Analytics**: GET /api/analytics/votes

## User Interface

### Vote Buttons
- **Layout**: Horizontal button group below search results
- **States**: Default, active (voted), disabled (loading)
- **Feedback**: Visual confirmation of vote submission
- **Counts**: Number display next to each category

### Vote Display
- **Bar Charts**: Visual representation of vote distribution
- **Percentages**: Relative vote share per category
- **Total Votes**: Overall community participation metric

## Technical Details

### Vote Submission
- **Validation**: Ensures valid search_id and vote_id
- **IP Tracking**: Records voter IP for analytics (not public)
- **Duplicate Prevention**: Users can vote once per search
- **Comments**: Optional text feedback from voters

### Data Processing
- **Real-time Updates**: Vote counts refresh after submission
- **Aggregation**: Efficient counting via SQL queries
- **Analytics**: Geographic and temporal vote patterns

### Security Measures
- **Rate Limiting**: Prevents spam voting
- **IP Validation**: Basic abuse prevention
- **Input Sanitization**: Comment text cleaned for safety

## Analytics Integration

### Vote Patterns
- **Geographic Distribution**: Voting patterns by country/region
- **Temporal Trends**: Vote submissions over time
- **Category Popularity**: Most common vote types

### Censorship Indicators
- **Censored vs Uncensored**: Primary censorship metric
- **Regional Variations**: How voting patterns differ geographically
- **Translation Quality**: Community assessment of translation accuracy

### Dashboard Metrics
- **Total Votes**: Overall community participation
- **Vote Velocity**: Votes per day/week/month
- **Category Breakdown**: Pie charts of vote distribution

## User Experience

### Voting Flow
1. User views search comparison results
2. Evaluates content difference between engines
3. Selects appropriate vote category
4. Sees immediate feedback and updated counts
5. Can view aggregate voting patterns

### Accessibility
- **Keyboard Navigation**: Tab-through vote buttons
- **Screen Readers**: ARIA labels for vote categories
- **Color Coding**: Not sole indicator of vote status
- **Clear Labels**: Descriptive button text

## Community Moderation

### Spam Prevention
- **IP Rate Limiting**: Maximum votes per IP per time period
- **Suspicious Pattern Detection**: Automated flagging of unusual voting
- **Manual Review**: Admin tools for reviewing questionable votes

### Quality Control
- **Vote Validation**: Checks for reasonable vote patterns
- **Comment Moderation**: Optional review of user comments
- **Category Balance**: Monitoring for vote category bias

## Future Enhancements

### Enhanced Categories
- **Subcategories**: More granular voting options
- **Custom Tags**: User-defined classification labels
- **Severity Levels**: Degrees of censorship/quality

### Advanced Features
- **Vote Explanations**: Detailed reasoning for votes
- **User Reputation**: Weighted voting based on history
- **Expert Validation**: Verified expert vote distinction

### Analytics Improvements
- **Machine Learning**: Pattern detection in vote data
- **Predictive Models**: Anticipate censorship trends
- **Cross-Platform**: Compare with other censorship datasets

---

*For search comparison context, see [Search Comparison](./SEARCH-COMPARISON.md)*
*For analytics integration, see [Analytics Dashboard](./ANALYTICS.md)*