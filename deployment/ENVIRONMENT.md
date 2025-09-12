# Environment Configuration Guide

This guide covers environment variable management, configuration patterns, and environment-specific setups for Firewall Cafe across development, staging, and production environments.

## Table of Contents
- [Environment Overview](#environment-overview)
- [Configuration Management](#configuration-management)
- [Environment Variables](#environment-variables)
- [Security Best Practices](#security-best-practices)
- [Environment-Specific Setup](#environment-specific-setup)
- [Configuration Templates](#configuration-templates)
- [Troubleshooting](#troubleshooting)

## Environment Overview

### Environment Types

#### Development
- **Purpose**: Local development and testing
- **Database**: Local PostgreSQL instance
- **APIs**: Development keys with higher limits
- **Logging**: Debug level, console output
- **Security**: Relaxed for development efficiency

#### Staging  
- **Purpose**: Pre-production testing and validation
- **Database**: Cloud SQL with production-like data
- **APIs**: Production keys with rate limits
- **Logging**: Info level, cloud logging
- **Security**: Production-like security measures

#### Production
- **Purpose**: Live application serving users
- **Database**: Production Cloud SQL with backups
- **APIs**: Production keys with monitoring
- **Logging**: Warn/error level, comprehensive monitoring
- **Security**: Full security hardening

### Configuration Sources Priority
1. Environment variables (highest priority)
2. `.env` files
3. Configuration files (`config.js`)
4. Default values (lowest priority)

## Configuration Management

### Client-Side Configuration

#### Environment Files
```bash
# Development
client/.env.development

# Production  
client/.env.production

# Local overrides (not committed)
client/.env.local
```

#### React App Variables
All client-side environment variables must start with `REACT_APP_`:

```bash
# Frontend Configuration
REACT_APP_API_URL=http://localhost:8080
REACT_APP_BACKEND_API_URL=http://localhost:11458
REACT_APP_DEBUG_MODE=true
REACT_APP_ENABLE_ANALYTICS=true

# Feature Flags
REACT_APP_ENABLE_VOTING=true
REACT_APP_ENABLE_HEATMAPS=true
REACT_APP_ENABLE_EXPERT_COMMENTARY=true

# External Services
REACT_APP_GOOGLE_TRANSLATE_API_KEY=your_key_here
```

### Server-Side Configuration

#### Express Proxy Server (`client/index.js`)
```javascript
// client/server/config.js
const config = {
  port: process.env.PORT || 8080,
  apiUrl: process.env.API_URL || 'http://localhost:11458',
  corsOrigins: process.env.CORS_ORIGINS?.split(',') || ['http://localhost:3000'],
  enableCache: process.env.ENABLE_CACHE === 'true',
  cacheTimeout: parseInt(process.env.CACHE_TIMEOUT) || 300000, // 5 minutes
  
  // Logging
  logLevel: process.env.LOG_LEVEL || 'info',
  enableRequestLogging: process.env.ENABLE_REQUEST_LOGGING === 'true',
  
  // Security
  enableRateLimiting: process.env.ENABLE_RATE_LIMITING === 'true',
  maxRequestsPerMinute: parseInt(process.env.MAX_REQUESTS_PER_MINUTE) || 100,
};

module.exports = config;
```

#### Backend API Server (`server/api/config.js`)
```javascript
// server/api/config.js
const config = {
  // Database Configuration
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT) || 5432,
    user: process.env.DB_USER || 'firewallcafe',
    password: process.env.DB_PASSWORD || 'password',
    database: process.env.DB_NAME || 'firewallcafe',
    ssl: process.env.DB_SSL === 'true' ? { rejectUnauthorized: false } : false,
    max: parseInt(process.env.DB_POOL_MAX) || 10, // Connection pool
    idleTimeoutMillis: parseInt(process.env.DB_IDLE_TIMEOUT) || 30000,
  },
  
  // API Keys
  serperApiKey: process.env.SERPER_API_KEY,
  googleTranslateApiKey: process.env.GOOGLE_TRANSLATE_API_KEY,
  baiduApiKey: process.env.BAIDU_API_KEY,
  
  // Server Configuration
  port: parseInt(process.env.API_PORT) || 11458,
  corsOrigins: process.env.CORS_ORIGINS?.split(',') || ['*'],
  
  // Security
  sharedSecret: process.env.SHARED_SECRET || 'dev-secret-change-in-production',
  enableAuth: process.env.ENABLE_AUTH !== 'false',
  
  // Features
  enableVoting: process.env.ENABLE_VOTING !== 'false',
  enableAnalytics: process.env.ENABLE_ANALYTICS !== 'false',
  enableGeolocation: process.env.ENABLE_GEOLOCATION !== 'false',
  
  // Rate Limiting
  rateLimitWindow: parseInt(process.env.RATE_LIMIT_WINDOW) || 60000, // 1 minute
  rateLimitMax: parseInt(process.env.RATE_LIMIT_MAX) || 100,
  
  // Caching
  enableCache: process.env.ENABLE_CACHE === 'true',
  cacheTimeout: parseInt(process.env.CACHE_TIMEOUT) || 300000, // 5 minutes
  redisCacheUrl: process.env.REDIS_CACHE_URL, // Optional Redis caching
  
  // Logging
  logLevel: process.env.LOG_LEVEL || 'info',
  enableAccessLogging: process.env.ENABLE_ACCESS_LOGGING !== 'false',
  
  // External Services
  ipGeolocationProvider: process.env.IP_GEOLOCATION_PROVIDER || 'ipapi',
  ipGeolocationApiKey: process.env.IP_GEOLOCATION_API_KEY,
};

module.exports = config;
```

## Environment Variables

### Required Variables

#### Development Environment
```bash
# Database
DB_HOST=localhost
DB_PORT=5432
DB_USER=firewallcafe
DB_PASSWORD=your_password
DB_NAME=firewallcafe

# API Keys (development/testing)
SERPER_API_KEY=your_dev_serper_key
GOOGLE_TRANSLATE_API_KEY=your_dev_translate_key

# Security (use secure random in production)
SHARED_SECRET=dev-secret-only

# Features (enable all for development)
ENABLE_VOTING=true
ENABLE_ANALYTICS=true
ENABLE_GEOLOCATION=true
```

#### Staging Environment
```bash
# Database (Cloud SQL)
DB_HOST=/cloudsql/firewall-cafe-staging:us-central1:firewall-db
DB_USER=firewallcafe
DB_PASSWORD=secure_staging_password
DB_NAME=firewallcafe
DB_SSL=true

# API Keys (production keys with limits)
SERPER_API_KEY=your_prod_serper_key
GOOGLE_TRANSLATE_API_KEY=your_prod_translate_key

# Security
SHARED_SECRET=secure_staging_secret

# Environment
NODE_ENV=staging
LOG_LEVEL=info

# Features
ENABLE_VOTING=true
ENABLE_ANALYTICS=true
ENABLE_GEOLOCATION=true

# Performance
ENABLE_CACHE=true
CACHE_TIMEOUT=300000
DB_POOL_MAX=5
```

#### Production Environment
```bash
# Database (Cloud SQL)
DB_HOST=/cloudsql/firewall-cafe-prod:us-central1:firewall-db
DB_USER=firewallcafe
DB_PASSWORD=secure_production_password
DB_NAME=firewallcafe
DB_SSL=true

# API Keys
SERPER_API_KEY=your_prod_serper_key
GOOGLE_TRANSLATE_API_KEY=your_prod_translate_key

# Security
SHARED_SECRET=highly_secure_production_secret
ENABLE_AUTH=true

# Environment
NODE_ENV=production
LOG_LEVEL=warn

# Performance & Scaling
ENABLE_CACHE=true
CACHE_TIMEOUT=600000  # 10 minutes in production
DB_POOL_MAX=20
RATE_LIMIT_MAX=50     # Stricter rate limiting

# Monitoring
ENABLE_ACCESS_LOGGING=true
ENABLE_REQUEST_LOGGING=false  # Reduce log volume
```

### Optional Variables

#### Feature Flags
```bash
# Experimental Features
ENABLE_US_STATES_HEATMAP=true
ENABLE_EXPERT_COMMENTARY=true
ENABLE_ADVANCED_FILTERS=true
ENABLE_CSV_EXPORT=true

# UI Features  
ENABLE_DARK_MODE=true
ENABLE_MOBILE_OPTIMIZATIONS=true
ENABLE_KEYBOARD_SHORTCUTS=true

# Performance Features
ENABLE_IMAGE_LAZY_LOADING=true
ENABLE_COMPONENT_LAZY_LOADING=true
ENABLE_SERVICE_WORKER=true
```

#### Third-Party Integrations
```bash
# Analytics
GOOGLE_ANALYTICS_ID=UA-XXXXXXXX-X
MIXPANEL_PROJECT_TOKEN=your_mixpanel_token

# Error Tracking
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
BUGSNAG_API_KEY=your_bugsnag_key

# Communication
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/xxx
EMAIL_SERVICE_API_KEY=your_email_service_key

# CDN & Storage
AWS_S3_BUCKET=firewall-images-prod
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret
CLOUDINARY_CLOUD_NAME=your_cloudinary_name
```

## Security Best Practices

### Secret Management

#### Google Cloud Secret Manager (Recommended)
```bash
# Store secrets
echo -n "actual_secret_value" | gcloud secrets create secret-name --data-file=-

# Access in application
const { SecretManagerServiceClient } = require('@google-cloud/secret-manager');
const client = new SecretManagerServiceClient();

async function getSecret(secretName) {
  const [version] = await client.accessSecretVersion({
    name: `projects/PROJECT_ID/secrets/${secretName}/versions/latest`,
  });
  return version.payload.data.toString('utf8');
}
```

#### Environment Variable Validation
```javascript
// config/validator.js
const requiredEnvVars = {
  development: ['DB_PASSWORD', 'SERPER_API_KEY'],
  staging: ['DB_PASSWORD', 'SERPER_API_KEY', 'SHARED_SECRET'],
  production: ['DB_PASSWORD', 'SERPER_API_KEY', 'SHARED_SECRET', 'GOOGLE_TRANSLATE_API_KEY']
};

function validateEnvironment() {
  const env = process.env.NODE_ENV || 'development';
  const required = requiredEnvVars[env] || [];
  
  const missing = required.filter(varName => !process.env[varName]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

module.exports = { validateEnvironment };
```

#### Security Headers Configuration
```javascript
// security/headers.js
const securityHeaders = {
  development: {
    'X-Content-Type-Options': 'nosniff',
    'X-Frame-Options': 'DENY',
    'X-XSS-Protection': '1; mode=block',
  },
  
  production: {
    'X-Content-Type-Options': 'nosniff',
    'X-Frame-Options': 'DENY',
    'X-XSS-Protection': '1; mode=block',
    'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
    'Content-Security-Policy': "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';",
    'Referrer-Policy': 'strict-origin-when-cross-origin',
  }
};

module.exports = securityHeaders;
```

### Access Control

#### API Key Rotation Strategy
```javascript
// config/apiKeys.js
const apiKeys = {
  // Primary keys (current)
  serper: process.env.SERPER_API_KEY,
  translate: process.env.GOOGLE_TRANSLATE_API_KEY,
  
  // Backup keys (for rotation)
  serperBackup: process.env.SERPER_API_KEY_BACKUP,
  translateBackup: process.env.GOOGLE_TRANSLATE_API_KEY_BACKUP,
};

// Key rotation logic
function getApiKey(service, useBackup = false) {
  const key = useBackup ? apiKeys[`${service}Backup`] : apiKeys[service];
  if (!key) {
    throw new Error(`API key not found for service: ${service}`);
  }
  return key;
}

module.exports = { getApiKey };
```

## Environment-Specific Setup

### Development Environment
```bash
# .env.development
NODE_ENV=development

# Database
DB_HOST=localhost
DB_PORT=5432
DB_USER=firewallcafe
DB_PASSWORD=dev_password
DB_NAME=firewallcafe

# React App
REACT_APP_API_URL=http://localhost:8080
REACT_APP_DEBUG_MODE=true
REACT_APP_ENABLE_HOT_RELOAD=true

# Features (enable all for testing)
ENABLE_VOTING=true
ENABLE_ANALYTICS=true
ENABLE_GEOLOCATION=true
ENABLE_EXPERT_COMMENTARY=true

# Logging
LOG_LEVEL=debug
ENABLE_ACCESS_LOGGING=true
ENABLE_REQUEST_LOGGING=true

# Performance (disabled for development)
ENABLE_CACHE=false
ENABLE_RATE_LIMITING=false
```

### Staging Environment
```bash
# .env.staging  
NODE_ENV=staging

# Database (Cloud SQL)
DB_HOST=/cloudsql/firewall-cafe-staging:us-central1:firewall-db
DB_USER=firewallcafe
DB_NAME=firewallcafe
DB_SSL=true

# React App
REACT_APP_API_URL=https://staging.firewallcafe.com
REACT_APP_DEBUG_MODE=false

# Security
ENABLE_AUTH=true

# Performance
ENABLE_CACHE=true
CACHE_TIMEOUT=300000
ENABLE_RATE_LIMITING=true
RATE_LIMIT_MAX=200

# Monitoring  
LOG_LEVEL=info
ENABLE_ACCESS_LOGGING=true
```

### Production Environment
```bash
# .env.production
NODE_ENV=production

# Database (Cloud SQL)
DB_HOST=/cloudsql/firewall-cafe-prod:us-central1:firewall-db
DB_USER=firewallcafe
DB_NAME=firewallcafe
DB_SSL=true
DB_POOL_MAX=20

# React App
REACT_APP_API_URL=https://firewallcafe.com
REACT_APP_DEBUG_MODE=false
REACT_APP_ENABLE_ANALYTICS=true

# Security (all enabled)
ENABLE_AUTH=true
SHARED_SECRET=production_secret_from_secret_manager

# Performance (optimized)
ENABLE_CACHE=true
CACHE_TIMEOUT=600000  # 10 minutes
ENABLE_RATE_LIMITING=true
RATE_LIMIT_MAX=50

# Monitoring (minimal logging)
LOG_LEVEL=warn
ENABLE_ACCESS_LOGGING=true
ENABLE_REQUEST_LOGGING=false

# External Services
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
GOOGLE_ANALYTICS_ID=UA-XXXXXXXX-X
```

## Configuration Templates

### Environment Template Generator
```javascript
// scripts/generate-env-template.js
const environments = {
  development: {
    NODE_ENV: 'development',
    DB_HOST: 'localhost',
    DB_PORT: '5432',
    DB_USER: 'firewallcafe',
    DB_PASSWORD: 'your_password',
    DB_NAME: 'firewallcafe',
    
    SERPER_API_KEY: 'your_dev_serper_key',
    GOOGLE_TRANSLATE_API_KEY: 'your_dev_translate_key',
    SHARED_SECRET: 'dev-secret-only',
    
    REACT_APP_API_URL: 'http://localhost:8080',
    REACT_APP_DEBUG_MODE: 'true',
    
    LOG_LEVEL: 'debug',
    ENABLE_CACHE: 'false',
    ENABLE_RATE_LIMITING: 'false',
  },
  
  production: {
    NODE_ENV: 'production',
    DB_HOST: '/cloudsql/firewall-cafe-prod:us-central1:firewall-db',
    DB_USER: 'firewallcafe', 
    DB_NAME: 'firewallcafe',
    DB_SSL: 'true',
    
    REACT_APP_API_URL: 'https://firewallcafe.com',
    REACT_APP_DEBUG_MODE: 'false',
    
    LOG_LEVEL: 'warn',
    ENABLE_CACHE: 'true',
    ENABLE_RATE_LIMITING: 'true',
    ENABLE_AUTH: 'true',
  }
};

function generateEnvFile(environment) {
  const config = environments[environment];
  if (!config) {
    throw new Error(`Environment ${environment} not found`);
  }
  
  const envContent = Object.entries(config)
    .map(([key, value]) => `${key}=${value}`)
    .join('\n');
    
  return `# Generated ${environment} environment file\n# Generated on ${new Date().toISOString()}\n\n${envContent}`;
}

// Usage
console.log(generateEnvFile('development'));
```

### Configuration Validation
```javascript
// config/validation.js
const Joi = require('joi');

const configSchema = {
  development: Joi.object({
    DB_HOST: Joi.string().required(),
    DB_PORT: Joi.number().integer().min(1).max(65535).default(5432),
    DB_USER: Joi.string().required(),
    DB_PASSWORD: Joi.string().required(),
    SERPER_API_KEY: Joi.string().required(),
    LOG_LEVEL: Joi.string().valid('debug', 'info', 'warn', 'error').default('debug'),
  }),
  
  production: Joi.object({
    DB_HOST: Joi.string().required(),
    DB_USER: Joi.string().required(),
    DB_PASSWORD: Joi.string().required(),
    DB_SSL: Joi.boolean().default(true),
    SHARED_SECRET: Joi.string().min(32).required(),
    SERPER_API_KEY: Joi.string().required(),
    GOOGLE_TRANSLATE_API_KEY: Joi.string().required(),
    LOG_LEVEL: Joi.string().valid('info', 'warn', 'error').default('warn'),
  })
};

function validateConfig(environment = process.env.NODE_ENV) {
  const schema = configSchema[environment];
  if (!schema) {
    throw new Error(`No validation schema for environment: ${environment}`);
  }
  
  const { error, value } = schema.validate(process.env, { 
    allowUnknown: true,
    stripUnknown: false 
  });
  
  if (error) {
    throw new Error(`Configuration validation failed: ${error.message}`);
  }
  
  return value;
}

module.exports = { validateConfig };
```

## Troubleshooting

### Common Issues

#### 1. Missing Environment Variables
```bash
# Check if all required variables are set
node -e "
const required = ['DB_PASSWORD', 'SERPER_API_KEY'];
const missing = required.filter(v => !process.env[v]);
if (missing.length) {
  console.error('Missing:', missing.join(', '));
  process.exit(1);
} else {
  console.log('All required variables present');
}
"
```

#### 2. Database Connection Issues
```bash
# Test database connection with current environment
node -e "
const { Pool } = require('pg');
const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  ssl: process.env.DB_SSL === 'true'
});
pool.query('SELECT NOW()')
  .then(res => console.log('DB OK:', res.rows[0]))
  .catch(err => console.error('DB Error:', err.message))
  .finally(() => pool.end());
"
```

#### 3. API Key Validation
```bash
# Test Serper API key
curl -X GET "https://google.serper.dev/search?q=test" \
  -H "X-API-KEY: $SERPER_API_KEY" \
  -H "Content-Type: application/json"

# Test Google Translate API
curl -X POST "https://translation.googleapis.com/language/translate/v2?key=$GOOGLE_TRANSLATE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"q":"Hello","source":"en","target":"zh"}'
```

### Debug Commands
```bash
# Print all environment variables (development only!)
printenv | grep -E '^(REACT_APP_|DB_|API_|ENABLE_)' | sort

# Validate configuration
node -e "require('./config/validation').validateConfig()"

# Test specific environment
NODE_ENV=staging node -e "console.log(require('./config'))"

# Check database connection
node -e "require('./config/database').testConnection()"
```

### Configuration Checklist

#### Before Deployment
- [ ] All required environment variables set
- [ ] API keys valid and have sufficient quotas
- [ ] Database connection tested
- [ ] Secrets stored securely (Secret Manager)
- [ ] CORS origins configured correctly
- [ ] Rate limiting configured appropriately
- [ ] Logging level appropriate for environment
- [ ] Security headers configured
- [ ] SSL/TLS certificates valid

#### After Deployment
- [ ] Application starts without errors
- [ ] Database queries executing successfully
- [ ] API endpoints responding correctly
- [ ] Frontend loads and functions properly
- [ ] External API integrations working
- [ ] Monitoring and alerting configured
- [ ] Log aggregation working
- [ ] Performance metrics within acceptable ranges

---

*For deployment-specific configuration, see [Google Cloud Deployment Guide](./GOOGLE-CLOUD.md)*
*For security hardening, see [Security Best Practices](./SECURITY.md)*