# Monitoring and Logging Setup

This guide covers comprehensive monitoring, logging, and alerting setup for Firewall Cafe using Google Cloud Platform services and third-party tools.

## Table of Contents
- [Monitoring Overview](#monitoring-overview)
- [Google Cloud Monitoring](#google-cloud-monitoring)
- [Application Logging](#application-logging)
- [Performance Monitoring](#performance-monitoring)
- [Error Tracking](#error-tracking)
- [Alerting Setup](#alerting-setup)
- [Health Checks](#health-checks)
- [Dashboards](#dashboards)
- [Log Analysis](#log-analysis)
- [Troubleshooting](#troubleshooting)

## Monitoring Overview

### Key Metrics to Track

#### Application Performance
- **Response Time**: API endpoint latency (95th percentile)
- **Throughput**: Requests per second
- **Error Rate**: 4xx/5xx error percentage
- **Availability**: Uptime percentage

#### Infrastructure Health
- **CPU Utilization**: App Engine instance CPU usage
- **Memory Usage**: Memory consumption patterns
- **Database Performance**: Query response time, connection pool
- **Network**: Ingress/egress traffic patterns

#### Business Metrics
- **Search Success Rate**: Successful Google/Baidu comparisons
- **User Engagement**: Active users, session duration
- **Data Quality**: Translation accuracy, vote submissions
- **Geographic Distribution**: Search patterns by location

### Monitoring Stack
- **Google Cloud Monitoring**: Infrastructure and application metrics
- **Google Cloud Logging**: Centralized log management
- **Cloud Trace**: Request tracing and latency analysis
- **Cloud Profiler**: Performance profiling
- **Sentry** (Optional): Error tracking and monitoring
- **Uptime Robot** (Optional): External uptime monitoring

## Google Cloud Monitoring

### Enable Monitoring APIs
```bash
# Enable required APIs
gcloud services enable monitoring.googleapis.com
gcloud services enable logging.googleapis.com
gcloud services enable cloudtrace.googleapis.com
gcloud services enable cloudprofiler.googleapis.com
```

### Basic Metrics Configuration
```javascript
// monitoring/metrics.js
const monitoring = require('@google-cloud/monitoring');
const client = new monitoring.MetricServiceClient();

// Custom metrics
const customMetrics = {
  searchSuccessRate: 'custom.googleapis.com/firewall/search_success_rate',
  translationAccuracy: 'custom.googleapis.com/firewall/translation_accuracy',
  userSessions: 'custom.googleapis.com/firewall/active_sessions',
  voteSubmissions: 'custom.googleapis.com/firewall/vote_submissions'
};

async function recordMetric(metricType, value, labels = {}) {
  const projectId = process.env.GOOGLE_CLOUD_PROJECT;
  
  const dataPoint = {
    interval: {
      endTime: { seconds: Date.now() / 1000 }
    },
    value: { doubleValue: value }
  };
  
  const timeSeriesData = {
    metric: {
      type: metricType,
      labels: labels
    },
    resource: {
      type: 'gae_app',
      labels: {
        project_id: projectId,
        module_id: 'default'
      }
    },
    points: [dataPoint]
  };
  
  await client.createTimeSeries({
    name: `projects/${projectId}`,
    timeSeries: [timeSeriesData]
  });
}

// Usage examples
async function trackSearchAttempt(success, engine, location) {
  await recordMetric(
    customMetrics.searchSuccessRate,
    success ? 1 : 0,
    { engine, location, success: success.toString() }
  );
}

module.exports = { recordMetric, trackSearchAttempt, customMetrics };
```

### Alert Policies Configuration
```yaml
# alerting-policies.yaml
displayName: "High Error Rate"
documentation:
  content: "Error rate is above 5% for more than 5 minutes"
conditions:
  - displayName: "Error rate condition"
    conditionThreshold:
      filter: 'resource.type="gae_app" AND metric.type="appengine.googleapis.com/http/server/response_count"'
      comparison: COMPARISON_GREATER_THAN
      thresholdValue: 0.05
      duration: "300s"
      aggregations:
        - alignmentPeriod: "60s"
          perSeriesAligner: ALIGN_RATE
          crossSeriesReducer: REDUCE_SUM
          groupByFields:
            - "resource.project_id"
            - "resource.module_id"
alertStrategy:
  autoClose: "1800s"
notificationChannels:
  - "projects/PROJECT_ID/notificationChannels/CHANNEL_ID"
enabled: true
```

Create alert policies:
```bash
# Create alert policy from YAML
gcloud alpha monitoring policies create --policy-from-file=alerting-policies.yaml
```

## Application Logging

### Structured Logging Setup
```javascript
// logging/logger.js
const { Logging } = require('@google-cloud/logging');
const logging = new Logging();
const log = logging.log('firewall-cafe');

const LogLevel = {
  DEBUG: 'DEBUG',
  INFO: 'INFO', 
  WARN: 'WARN',
  ERROR: 'ERROR'
};

class Logger {
  constructor(component = 'app') {
    this.component = component;
  }
  
  log(level, message, metadata = {}) {
    const logEntry = log.entry({
      severity: level,
      timestamp: new Date(),
      labels: {
        component: this.component,
        environment: process.env.NODE_ENV || 'development'
      }
    }, {
      message,
      ...metadata,
      // Add request context if available
      ...(metadata.req && this.extractRequestContext(metadata.req))
    });
    
    // In development, also log to console
    if (process.env.NODE_ENV === 'development') {
      console.log(`[${level}] ${this.component}: ${message}`, metadata);
    }
    
    log.write(logEntry);
  }
  
  extractRequestContext(req) {
    return {
      httpRequest: {
        requestMethod: req.method,
        requestUrl: req.url,
        userAgent: req.get('User-Agent'),
        remoteIp: req.ip,
        referer: req.get('Referer')
      },
      userId: req.user?.id,
      sessionId: req.sessionID
    };
  }
  
  debug(message, metadata) { this.log(LogLevel.DEBUG, message, metadata); }
  info(message, metadata) { this.log(LogLevel.INFO, message, metadata); }
  warn(message, metadata) { this.log(LogLevel.WARN, message, metadata); }
  error(message, metadata) { this.log(LogLevel.ERROR, message, metadata); }
}

// Component-specific loggers
const apiLogger = new Logger('api');
const searchLogger = new Logger('search');
const analyticsLogger = new Logger('analytics');

module.exports = { Logger, apiLogger, searchLogger, analyticsLogger };
```

### Request Logging Middleware
```javascript
// middleware/requestLogger.js
const { apiLogger } = require('../logging/logger');
const { v4: uuidv4 } = require('uuid');

function requestLogger(req, res, next) {
  const startTime = Date.now();
  const requestId = uuidv4();
  
  // Add request ID to request object
  req.requestId = requestId;
  
  // Log request start
  apiLogger.info('Request started', {
    requestId,
    method: req.method,
    url: req.url,
    userAgent: req.get('User-Agent'),
    ip: req.ip,
    req: req
  });
  
  // Override res.end to log response
  const originalEnd = res.end;
  res.end = function(chunk, encoding) {
    const duration = Date.now() - startTime;
    
    apiLogger.info('Request completed', {
      requestId,
      statusCode: res.statusCode,
      duration,
      contentLength: res.get('Content-Length'),
      req: req
    });
    
    // Call original end method
    originalEnd.call(res, chunk, encoding);
  };
  
  next();
}

module.exports = requestLogger;
```

### Error Logging
```javascript
// middleware/errorLogger.js  
const { apiLogger } = require('../logging/logger');

function errorLogger(err, req, res, next) {
  const errorContext = {
    error: {
      name: err.name,
      message: err.message,
      stack: err.stack
    },
    requestId: req.requestId,
    method: req.method,
    url: req.url,
    userId: req.user?.id,
    req: req
  };
  
  // Log different severity based on error type
  if (err.statusCode && err.statusCode < 500) {
    apiLogger.warn('Client error', errorContext);
  } else {
    apiLogger.error('Server error', errorContext);
  }
  
  next(err);
}

module.exports = errorLogger;
```

### Search Activity Logging
```javascript
// logging/searchLogger.js
const { searchLogger } = require('./logger');

class SearchActivityLogger {
  static logSearchAttempt(query, engines, user, location) {
    searchLogger.info('Search attempt', {
      query: query.substring(0, 100), // Truncate long queries
      engines: engines,
      userId: user?.id || 'anonymous',
      location: location,
      timestamp: new Date().toISOString(),
      type: 'search_attempt'
    });
  }
  
  static logSearchSuccess(query, results, duration) {
    searchLogger.info('Search completed', {
      query: query.substring(0, 100),
      googleResults: results.google?.length || 0,
      baiduResults: results.baidu?.length || 0,
      duration: duration,
      timestamp: new Date().toISOString(),
      type: 'search_success'
    });
  }
  
  static logSearchError(query, error, engine) {
    searchLogger.error('Search failed', {
      query: query.substring(0, 100),
      engine: engine,
      error: {
        message: error.message,
        type: error.constructor.name
      },
      timestamp: new Date().toISOString(),
      type: 'search_error'
    });
  }
  
  static logTranslationAttempt(originalText, targetLanguage, result) {
    searchLogger.info('Translation attempt', {
      originalText: originalText.substring(0, 100),
      targetLanguage: targetLanguage,
      success: !!result,
      translatedText: result ? result.substring(0, 100) : null,
      timestamp: new Date().toISOString(),
      type: 'translation'
    });
  }
  
  static logVoteSubmission(searchId, voteType, userId) {
    searchLogger.info('Vote submitted', {
      searchId: searchId,
      voteType: voteType,
      userId: userId || 'anonymous',
      timestamp: new Date().toISOString(),
      type: 'vote_submission'
    });
  }
}

module.exports = SearchActivityLogger;
```

## Performance Monitoring

### Cloud Trace Integration
```javascript
// monitoring/trace.js
const tracer = require('@google-cloud/trace-agent').start({
  projectId: process.env.GOOGLE_CLOUD_PROJECT,
  samplingRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0 // 10% sampling in prod
});

// Custom span creation for critical operations
function createSpan(name, operation) {
  return tracer.createChildSpan({ name }, async (span) => {
    try {
      const result = await operation();
      span.addLabel('success', 'true');
      return result;
    } catch (error) {
      span.addLabel('success', 'false');
      span.addLabel('error', error.message);
      throw error;
    } finally {
      span.endSpan();
    }
  });
}

// Usage in search operations
async function performSearch(query, engines) {
  return createSpan('search-operation', async () => {
    const results = {};
    
    if (engines.includes('google')) {
      results.google = await createSpan('google-search', () => 
        searchGoogle(query)
      );
    }
    
    if (engines.includes('baidu')) {
      results.baidu = await createSpan('baidu-search', () => 
        searchBaidu(query)
      );
    }
    
    return results;
  });
}

module.exports = { createSpan, performSearch };
```

### Performance Metrics Collection
```javascript
// monitoring/performance.js
const { recordMetric } = require('./metrics');

class PerformanceMonitor {
  static trackResponseTime(endpoint, duration) {
    recordMetric(
      'custom.googleapis.com/firewall/response_time',
      duration,
      { endpoint }
    );
  }
  
  static trackDatabaseQuery(query, duration, success) {
    recordMetric(
      'custom.googleapis.com/firewall/db_query_time',
      duration,
      { query: query.substring(0, 50), success: success.toString() }
    );
  }
  
  static trackSearchAPICall(provider, duration, success) {
    recordMetric(
      'custom.googleapis.com/firewall/search_api_time',
      duration,
      { provider, success: success.toString() }
    );
  }
  
  static trackMemoryUsage() {
    const usage = process.memoryUsage();
    recordMetric(
      'custom.googleapis.com/firewall/memory_usage',
      usage.heapUsed,
      { type: 'heap_used' }
    );
  }
}

// Middleware to track endpoint performance
function performanceTracker(req, res, next) {
  const startTime = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    PerformanceMonitor.trackResponseTime(req.route?.path || req.path, duration);
  });
  
  next();
}

module.exports = { PerformanceMonitor, performanceTracker };
```

## Error Tracking

### Sentry Integration (Optional)
```javascript
// monitoring/sentry.js
const Sentry = require('@sentry/node');
const { Integrations } = require('@sentry/tracing');

if (process.env.SENTRY_DSN) {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    integrations: [
      new Sentry.Integrations.Http({ tracing: true }),
      new Integrations.Express({ app }),
    ],
    tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
    
    beforeSend(event, hint) {
      // Filter out noise
      if (event.exception) {
        const error = hint.originalException;
        if (error && error.code === 'ECONNRESET') {
          return null; // Don't send connection reset errors
        }
      }
      return event;
    }
  });
}

// Express middleware
function sentryMiddleware(app) {
  if (process.env.SENTRY_DSN) {
    app.use(Sentry.Handlers.requestHandler());
    app.use(Sentry.Handlers.tracingHandler());
    
    // Error handler (must be last)
    app.use(Sentry.Handlers.errorHandler());
  }
}

module.exports = { sentryMiddleware };
```

### Custom Error Categories
```javascript
// errors/categorized.js
class SearchError extends Error {
  constructor(message, provider, query) {
    super(message);
    this.name = 'SearchError';
    this.provider = provider;
    this.query = query;
    this.category = 'search';
  }
}

class DatabaseError extends Error {
  constructor(message, query) {
    super(message);
    this.name = 'DatabaseError';
    this.query = query;
    this.category = 'database';
  }
}

class ValidationError extends Error {
  constructor(message, field, value) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
    this.value = value;
    this.category = 'validation';
  }
}

class RateLimitError extends Error {
  constructor(message, limit, window) {
    super(message);
    this.name = 'RateLimitError';
    this.limit = limit;
    this.window = window;
    this.category = 'rate_limit';
  }
}

module.exports = {
  SearchError,
  DatabaseError,
  ValidationError,
  RateLimitError
};
```

## Alerting Setup

### Alert Policy Creation
```bash
# Create notification channel (email)
gcloud alpha monitoring channels create \
  --display-name="Firewall Team" \
  --type=email \
  --channel-labels=email_address=alerts@firewallcafe.com

# Create alert policy for high error rate
gcloud alpha monitoring policies create \
  --policy-from-file=alerts/high-error-rate.yaml

# Create alert policy for database issues
gcloud alpha monitoring policies create \
  --policy-from-file=alerts/database-issues.yaml

# Create alert policy for API failures
gcloud alpha monitoring policies create \
  --policy-from-file=alerts/api-failures.yaml
```

### Critical Alerts Configuration
```yaml
# alerts/critical-alerts.yaml
displayName: "Critical System Failure"
documentation:
  content: "Multiple systems failing simultaneously"
combiner: OR
conditions:
  - displayName: "App Engine down"
    conditionAbsent:
      filter: 'resource.type="gae_app" AND metric.type="appengine.googleapis.com/http/server/response_count"'
      duration: "300s"
      
  - displayName: "Database connection failure"
    conditionThreshold:
      filter: 'resource.type="cloudsql_database" AND metric.type="cloudsql.googleapis.com/database/up"'
      comparison: COMPARISON_LESS_THAN
      thresholdValue: 1
      duration: "120s"
      
  - displayName: "Search API failure rate > 50%"
    conditionThreshold:
      filter: 'metric.type="custom.googleapis.com/firewall/search_success_rate"'
      comparison: COMPARISON_LESS_THAN
      thresholdValue: 0.5
      duration: "300s"

alertStrategy:
  autoClose: "3600s"
enabled: true
```

### Slack Integration
```javascript
// alerting/slack.js
const axios = require('axios');

class SlackAlerts {
  constructor(webhookUrl) {
    this.webhookUrl = webhookUrl;
  }
  
  async sendAlert(severity, title, message, metadata = {}) {
    const color = this.getSeverityColor(severity);
    
    const payload = {
      username: 'Firewall Monitoring',
      icon_emoji: ':rotating_light:',
      attachments: [{
        color: color,
        title: title,
        text: message,
        fields: [
          {
            title: 'Environment',
            value: process.env.NODE_ENV || 'unknown',
            short: true
          },
          {
            title: 'Timestamp',
            value: new Date().toISOString(),
            short: true
          },
          ...Object.entries(metadata).map(([key, value]) => ({
            title: key,
            value: value.toString(),
            short: true
          }))
        ],
        footer: 'Firewall Cafe Monitoring',
        ts: Math.floor(Date.now() / 1000)
      }]
    };
    
    try {
      await axios.post(this.webhookUrl, payload);
    } catch (error) {
      console.error('Failed to send Slack alert:', error.message);
    }
  }
  
  getSeverityColor(severity) {
    switch (severity.toLowerCase()) {
      case 'critical': return 'danger';
      case 'warning': return 'warning';
      case 'info': return 'good';
      default: return '#439FE0';
    }
  }
  
  async alertHighErrorRate(errorRate, duration) {
    await this.sendAlert(
      'critical',
      'High Error Rate Detected',
      `Error rate has reached ${(errorRate * 100).toFixed(1)}% for ${duration} minutes`,
      { 'Error Rate': `${(errorRate * 100).toFixed(1)}%`, Duration: `${duration}m` }
    );
  }
  
  async alertDatabaseIssue(error) {
    await this.sendAlert(
      'critical',
      'Database Connection Issue',
      `Database connection failed: ${error.message}`,
      { 'Error Type': error.constructor.name }
    );
  }
}

// Usage
const slack = process.env.SLACK_WEBHOOK_URL ? 
  new SlackAlerts(process.env.SLACK_WEBHOOK_URL) : null;

module.exports = { SlackAlerts, slack };
```

## Health Checks

### Application Health Check Endpoint
```javascript
// health/check.js
const { Pool } = require('pg');
const axios = require('axios');

class HealthChecker {
  constructor(config) {
    this.dbPool = new Pool(config.database);
    this.config = config;
  }
  
  async checkDatabase() {
    try {
      const start = Date.now();
      await this.dbPool.query('SELECT 1');
      const duration = Date.now() - start;
      
      return {
        status: 'healthy',
        responseTime: duration,
        timestamp: new Date().toISOString()
      };
    } catch (error) {
      return {
        status: 'unhealthy',
        error: error.message,
        timestamp: new Date().toISOString()
      };
    }
  }
  
  async checkSearchAPIs() {
    const results = {};
    
    // Check Serper API
    try {
      const start = Date.now();
      await axios.get('https://google.serper.dev/search?q=test', {
        headers: { 'X-API-KEY': this.config.serperApiKey },
        timeout: 5000
      });
      results.serper = {
        status: 'healthy',
        responseTime: Date.now() - start
      };
    } catch (error) {
      results.serper = {
        status: 'unhealthy',
        error: error.message
      };
    }
    
    // Check Google Translate API
    try {
      const start = Date.now();
      await axios.post(
        `https://translation.googleapis.com/language/translate/v2?key=${this.config.googleTranslateApiKey}`,
        { q: 'test', target: 'zh' },
        { timeout: 5000 }
      );
      results.googleTranslate = {
        status: 'healthy',
        responseTime: Date.now() - start
      };
    } catch (error) {
      results.googleTranslate = {
        status: 'unhealthy',
        error: error.message
      };
    }
    
    return results;
  }
  
  async getSystemInfo() {
    const usage = process.memoryUsage();
    return {
      nodeVersion: process.version,
      uptime: process.uptime(),
      memory: {
        used: usage.heapUsed,
        total: usage.heapTotal,
        external: usage.external
      },
      cpu: process.cpuUsage(),
      timestamp: new Date().toISOString()
    };
  }
  
  async performHealthCheck() {
    const results = {
      status: 'healthy',
      timestamp: new Date().toISOString(),
      services: {},
      system: {}
    };
    
    // Check database
    results.services.database = await this.checkDatabase();
    
    // Check external APIs
    results.services.apis = await this.checkSearchAPIs();
    
    // Get system info
    results.system = await this.getSystemInfo();
    
    // Determine overall status
    const unhealthyServices = Object.values(results.services)
      .flat()
      .filter(service => service.status === 'unhealthy');
      
    if (unhealthyServices.length > 0) {
      results.status = 'degraded';
    }
    
    // Critical services down
    if (results.services.database.status === 'unhealthy') {
      results.status = 'unhealthy';
    }
    
    return results;
  }
}

// Health check endpoint
function healthCheckRoute(req, res) {
  const checker = new HealthChecker(config);
  
  checker.performHealthCheck()
    .then(results => {
      const statusCode = results.status === 'unhealthy' ? 503 : 200;
      res.status(statusCode).json(results);
    })
    .catch(error => {
      res.status(500).json({
        status: 'error',
        error: error.message,
        timestamp: new Date().toISOString()
      });
    });
}

module.exports = { HealthChecker, healthCheckRoute };
```

### Liveness and Readiness Probes
```javascript
// health/probes.js
let isReady = false;
let isLive = true;

// Liveness probe - app is running
function livenessProbe(req, res) {
  if (isLive) {
    res.status(200).json({ status: 'alive' });
  } else {
    res.status(503).json({ status: 'not alive' });
  }
}

// Readiness probe - app is ready to serve traffic
function readinessProbe(req, res) {
  if (isReady) {
    res.status(200).json({ status: 'ready' });
  } else {
    res.status(503).json({ status: 'not ready' });
  }
}

// Initialize readiness after dependencies are ready
async function initializeReadiness() {
  try {
    // Check database connection
    await dbPool.query('SELECT 1');
    
    // Validate API keys
    if (!process.env.SERPER_API_KEY) {
      throw new Error('Serper API key not configured');
    }
    
    isReady = true;
    console.log('Application is ready to serve traffic');
  } catch (error) {
    console.error('Application failed readiness check:', error);
    isReady = false;
  }
}

module.exports = { livenessProbe, readinessProbe, initializeReadiness };
```

## Dashboards

### Google Cloud Monitoring Dashboard
```yaml
# dashboards/firewall-overview.yaml
displayName: "Firewall Cafe Overview"
gridLayout:
  widgets:
    - title: "Request Rate"
      xyChart:
        dataSets:
          - timeSeriesQuery:
              timeSeriesFilter:
                filter: 'resource.type="gae_app" metric.type="appengine.googleapis.com/http/server/request_count"'
                aggregation:
                  alignmentPeriod: "60s"
                  perSeriesAligner: "ALIGN_RATE"
        timeshiftDuration: "0s"
        yAxis:
          label: "Requests/sec"
          scale: "LINEAR"
          
    - title: "Error Rate"
      xyChart:
        dataSets:
          - timeSeriesQuery:
              timeSeriesFilter:
                filter: 'resource.type="gae_app" metric.type="appengine.googleapis.com/http/server/response_count" metric.label.response_code!="200"'
                aggregation:
                  alignmentPeriod: "60s"
                  perSeriesAligner: "ALIGN_RATE"
        yAxis:
          label: "Errors/sec"
          scale: "LINEAR"
          
    - title: "Response Latency"
      xyChart:
        dataSets:
          - timeSeriesQuery:
              timeSeriesFilter:
                filter: 'resource.type="gae_app" metric.type="appengine.googleapis.com/http/server/response_latencies"'
                aggregation:
                  alignmentPeriod: "60s"
                  perSeriesAligner: "ALIGN_PERCENTILE_95"
        yAxis:
          label: "Latency (ms)"
          scale: "LINEAR"
```

Create dashboard:
```bash
gcloud monitoring dashboards create --config-from-file=dashboards/firewall-overview.yaml
```

## Log Analysis

### Useful Log Queries
```bash
# Error logs from last hour
gcloud logging read "
  resource.type=gae_app 
  AND severity>=ERROR 
  AND timestamp>=\"2023-08-26T10:00:00Z\"
" --limit=50 --format=json

# Search-related logs
gcloud logging read "
  resource.type=gae_app 
  AND labels.component=\"search\"
  AND timestamp>=\"2023-08-26T10:00:00Z\"
" --limit=100

# Database connection errors
gcloud logging read "
  resource.type=gae_app 
  AND textPayload:\"connection\"
  AND severity>=WARNING
" --limit=20

# High response time requests
gcloud logging read "
  resource.type=gae_app 
  AND jsonPayload.duration>5000
" --limit=10
```

### Log-based Metrics
```bash
# Create log-based metric for search failures
gcloud logging metrics create search_failures \
  --description="Count of search API failures" \
  --log-filter='resource.type="gae_app" AND jsonPayload.type="search_error"'

# Create metric for high response times
gcloud logging metrics create slow_requests \
  --description="Count of requests taking >5 seconds" \
  --log-filter='resource.type="gae_app" AND jsonPayload.duration>5000'
```

## Troubleshooting

### Common Monitoring Issues

#### 1. Missing Metrics
```bash
# Check if monitoring agent is running
gcloud compute instances list --filter="status:RUNNING" --format="table(name,status)"

# Verify monitoring API is enabled
gcloud services list --enabled --filter="name:monitoring.googleapis.com"

# Test custom metrics
node -e "require('./monitoring/metrics').recordMetric('test_metric', 1)"
```

#### 2. Log Ingestion Issues
```bash
# Check logging API status
gcloud services list --enabled --filter="name:logging.googleapis.com"

# Test log writing
node -e "require('./logging/logger').apiLogger.info('Test log entry')"

# Check log router
gcloud logging sinks list
```

#### 3. Alert Policy Problems
```bash
# List all alert policies
gcloud alpha monitoring policies list

# Check notification channels
gcloud alpha monitoring channels list

# Test notification channel
gcloud alpha monitoring channels verify CHANNEL_ID
```

### Debug Commands
```bash
# Stream real-time logs
gcloud app logs tail -s default

# Check App Engine metrics
gcloud app versions list --service=default

# Monitor resource usage
gcloud monitoring metrics-descriptors list --filter="metric.type:appengine"

# Test health check endpoint
curl -I https://firewallcafe.com/health
```

---

*For deployment setup, see [Google Cloud Deployment Guide](./GOOGLE-CLOUD.md)*
*For backup procedures, see [Backup and Recovery Guide](./BACKUP.md)*