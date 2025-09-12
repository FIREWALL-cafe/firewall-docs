# Google Cloud Platform Deployment Guide

This guide covers deploying Firewall Cafe to Google Cloud Platform (GCP), including App Engine for the frontend and Cloud SQL for the database.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [Database Deployment](#database-deployment)
- [Application Deployment](#application-deployment)
- [Domain Configuration](#domain-configuration)
- [Security Setup](#security-setup)
- [Monitoring Setup](#monitoring-setup)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Accounts & Tools
1. **Google Cloud Account** with billing enabled
2. **Google Cloud CLI** installed and configured
3. **Domain name** (for production deployment)
4. **GitHub repository** access
5. **API keys** for external services

### Local Setup
```bash
# Install Google Cloud CLI
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# Initialize and authenticate
gcloud init
gcloud auth login
gcloud auth application-default login
```

### Required APIs
Enable the following GCP APIs:
```bash
gcloud services enable appengine.googleapis.com
gcloud services enable sqladmin.googleapis.com
gcloud services enable cloudbuild.googleapis.com
gcloud services enable secretmanager.googleapis.com
gcloud services enable logging.googleapis.com
gcloud services enable monitoring.googleapis.com
```

## Project Setup

### 1. Create GCP Project
```bash
# Create new project
gcloud projects create firewall-cafe-prod --name="Firewall Cafe Production"

# Set as current project
gcloud config set project firewall-cafe-prod

# Link billing account (required)
gcloud billing accounts list
gcloud billing projects link firewall-cafe-prod --billing-account=BILLING_ACCOUNT_ID
```

### 2. Initialize App Engine
```bash
# Initialize App Engine in your preferred region
gcloud app create --region=us-central1

# Verify App Engine setup
gcloud app describe
```

### 3. Project Configuration
Create `app.yaml` in your project root:
```yaml
# app.yaml
runtime: nodejs20

env_variables:
  NODE_ENV: production
  REACT_APP_API_URL: https://api.firewallcafe.com
  REACT_APP_BACKEND_API_URL: https://api.firewallcafe.com

automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.6

resources:
  cpu: 1
  memory_gb: 1
  disk_size_gb: 10

handlers:
  - url: /static
    static_dir: build/static
    secure: always

  - url: /.*
    script: auto
    secure: always

skip_files:
  - ^(.*/)?#.*#$
  - ^(.*/)?.*~$
  - ^(.*/)?.*\.py[co]$
  - ^(.*/)?.*/RCS/.*$
  - ^(.*/)?\..*$
  - node_modules/
  - src/
  - docs/
  - \.git/
```

## Database Deployment

### 1. Create Cloud SQL Instance
```bash
# Create PostgreSQL instance
gcloud sql instances create firewall-db \
  --database-version=POSTGRES_14 \
  --cpu=2 \
  --memory=4GB \
  --region=us-central1 \
  --storage-type=SSD \
  --storage-size=20GB \
  --backup \
  --maintenance-window-day=SUN \
  --maintenance-window-hour=06 \
  --maintenance-release-channel=production

# Set root password
gcloud sql users set-password postgres \
  --instance=firewall-db \
  --password=[SECURE_PASSWORD]
```

### 2. Create Database and User
```bash
# Create application database
gcloud sql databases create firewallcafe --instance=firewall-db

# Create application user
gcloud sql users create firewallcafe \
  --instance=firewall-db \
  --password=[APP_USER_PASSWORD]
```

### 3. Configure Database Access
```bash
# Get connection name
gcloud sql instances describe firewall-db --format="value(connectionName)"

# Allow App Engine access
gcloud sql instances patch firewall-db \
  --authorized-gae-apps=firewall-cafe-prod
```

### 4. Initialize Database Schema
```bash
# Connect to Cloud SQL Proxy locally
cloud_sql_proxy -instances=firewall-cafe-prod:us-central1:firewall-db=tcp:5432 &

# Run schema setup
cd server/api/db_resources
psql -h localhost -U firewallcafe -d firewallcafe -f config.sql

# Optional: Load test data
psql -h localhost -U firewallcafe -d firewallcafe -f test_inserts.sql
```

## Application Deployment

### 1. Prepare Environment Variables
Create `.env.production`:
```bash
# Database Configuration
DB_HOST=/cloudsql/firewall-cafe-prod:us-central1:firewall-db
DB_USER=firewallcafe
DB_PASSWORD=[APP_USER_PASSWORD]
DB_NAME=firewallcafe

# API Keys
SERPER_API_KEY=[YOUR_SERPER_KEY]
GOOGLE_TRANSLATE_API_KEY=[YOUR_TRANSLATE_KEY]

# Security
SHARED_SECRET=[YOUR_SHARED_SECRET]

# Frontend URLs
REACT_APP_API_URL=https://firewall-cafe-prod.ue.r.appspot.com
```

### 2. Use Secret Manager (Recommended)
```bash
# Store sensitive values in Secret Manager
echo -n "[DB_PASSWORD]" | gcloud secrets create db-password --data-file=-
echo -n "[SERPER_API_KEY]" | gcloud secrets create serper-api-key --data-file=-
echo -n "[SHARED_SECRET]" | gcloud secrets create shared-secret --data-file=-

# Grant App Engine access to secrets
gcloud secrets add-iam-policy-binding db-password \
  --member="serviceAccount:firewall-cafe-prod@appspot.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

Update `app.yaml` to use secrets:
```yaml
env_variables:
  NODE_ENV: production
  DB_HOST: /cloudsql/firewall-cafe-prod:us-central1:firewall-db
  DB_USER: firewallcafe
  DB_NAME: firewallcafe

beta_settings:
  cloud_sql_instances: firewall-cafe-prod:us-central1:firewall-db

automatic_scaling:
  min_instances: 1
  max_instances: 10

# Access secrets at runtime
includes:
  - env_variables.yaml  # Contains secret references
```

### 3. Build and Deploy
```bash
# Install dependencies and build
cd client
yarn install
yarn build

# Deploy to App Engine
gcloud app deploy --quiet

# Deploy backend API (if separate service)
cd ../server/api
gcloud app deploy api.yaml --quiet
```

### 4. Verify Deployment
```bash
# Get application URL
gcloud app browse

# Check logs
gcloud app logs tail -s default

# Test endpoints
curl https://firewall-cafe-prod.ue.r.appspot.com/api/dashboard
```

## Domain Configuration

### 1. Custom Domain Setup
```bash
# Add custom domain
gcloud app domain-mappings create firewallcafe.com

# Add subdomain for API
gcloud app domain-mappings create api.firewallcafe.com --service=api
```

### 2. SSL Certificate
```bash
# App Engine automatically provides managed SSL certificates
# Verify certificate status
gcloud app ssl-certificates list

# Check domain mapping
gcloud app domain-mappings list
```

### 3. DNS Configuration
Add these DNS records to your domain:
```
# A Records
firewallcafe.com       → 216.239.32.21
                         216.239.34.21
                         216.239.36.21
                         216.239.38.21

# AAAA Records (IPv6)
firewallcafe.com       → 2001:4860:4802:32::15
                         2001:4860:4802:34::15
                         2001:4860:4802:36::15
                         2001:4860:4802:38::15

# CNAME for subdomain
api.firewallcafe.com   → ghs.googlehosted.com
```

## Security Setup

### 1. Identity and Access Management (IAM)
```bash
# Create service account for application
gcloud iam service-accounts create firewall-app \
  --display-name="Firewall Cafe Application"

# Grant necessary permissions
gcloud projects add-iam-policy-binding firewall-cafe-prod \
  --member="serviceAccount:firewall-app@firewall-cafe-prod.iam.gserviceaccount.com" \
  --role="roles/cloudsql.client"

gcloud projects add-iam-policy-binding firewall-cafe-prod \
  --member="serviceAccount:firewall-app@firewall-cafe-prod.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

### 2. VPC Configuration (Optional)
```bash
# Create VPC for enhanced security
gcloud compute networks create firewall-vpc --subnet-mode=custom

# Create subnet
gcloud compute networks subnets create firewall-subnet \
  --network=firewall-vpc \
  --region=us-central1 \
  --range=10.0.0.0/24
```

### 3. Firewall Rules
```bash
# Configure Cloud SQL authorized networks (if not using VPC)
gcloud sql instances patch firewall-db \
  --authorized-networks=0.0.0.0/0  # Allow all (App Engine)

# For production, restrict to specific IP ranges
gcloud sql instances patch firewall-db \
  --authorized-networks="10.0.0.0/8,172.16.0.0/12"
```

## Monitoring Setup

### 1. Cloud Logging
```bash
# Configure log routing (automatic for App Engine)
# View logs
gcloud logging logs list

# Stream application logs
gcloud app logs tail -s default
gcloud app logs tail -s api
```

### 2. Cloud Monitoring
```bash
# Create notification channels
gcloud alpha monitoring channels create \
  --channel-content-from-file=notification-channel.json

# Create alerting policies
gcloud alpha monitoring policies create \
  --policy-from-file=alerting-policy.json
```

Example `alerting-policy.json`:
```json
{
  "displayName": "High Error Rate",
  "conditions": [
    {
      "displayName": "Error rate too high",
      "conditionThreshold": {
        "filter": "resource.type=\"gae_app\"",
        "comparison": "COMPARISON_GREATER_THAN",
        "thresholdValue": 0.05,
        "duration": "300s"
      }
    }
  ],
  "alertStrategy": {
    "autoClose": "1800s"
  },
  "enabled": true
}
```

### 3. Performance Monitoring
```bash
# Enable Cloud Trace
gcloud services enable cloudtrace.googleapis.com

# Enable Cloud Profiler  
gcloud services enable cloudprofiler.googleapis.com
```

## Environment Management

### 1. Multiple Environments
```bash
# Create staging project
gcloud projects create firewall-cafe-staging

# Set up staging environment
gcloud config configurations create staging
gcloud config set project firewall-cafe-staging
```

### 2. Environment-Specific Configs
Create separate `app.yaml` files:
```yaml
# app.staging.yaml
runtime: nodejs20
service: default

env_variables:
  NODE_ENV: staging
  REACT_APP_API_URL: https://staging-api.firewallcafe.com

# Reduced resources for staging
automatic_scaling:
  min_instances: 0
  max_instances: 2
```

### 3. Deployment Pipeline
```bash
# Deploy to staging
gcloud app deploy app.staging.yaml --project=firewall-cafe-staging

# Promote to production
gcloud app deploy app.yaml --project=firewall-cafe-prod
```

## CI/CD Setup

### 1. Cloud Build Configuration
Create `cloudbuild.yaml`:
```yaml
steps:
  # Install dependencies
  - name: 'node:20'
    entrypoint: 'yarn'
    args: ['install']
    dir: 'client'

  # Run tests
  - name: 'node:20'
    entrypoint: 'yarn'
    args: ['test:ci']
    dir: 'client'

  # Build application
  - name: 'node:20'
    entrypoint: 'yarn'
    args: ['build']
    dir: 'client'

  # Deploy to App Engine
  - name: 'gcr.io/cloud-builders/gcloud'
    args: ['app', 'deploy', '--quiet']
    dir: 'client'

timeout: 1200s
```

### 2. GitHub Integration
```bash
# Connect GitHub repository
gcloud builds triggers create github \
  --repo-name=firewall-cafe \
  --repo-owner=FIREWALL-cafe \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml
```

## Troubleshooting

### Common Issues

#### 1. Deployment Failures
```bash
# Check deployment status
gcloud app versions list

# View deployment logs
gcloud app logs tail -s default

# Check build logs
gcloud builds list --limit=5
gcloud builds log [BUILD_ID]
```

#### 2. Database Connection Issues
```bash
# Test Cloud SQL connectivity
gcloud sql connect firewall-db --user=firewallcafe

# Check authorized networks
gcloud sql instances describe firewall-db --format="value(settings.ipConfiguration.authorizedNetworks[].value)"

# Verify App Engine service account permissions
gcloud projects get-iam-policy firewall-cafe-prod
```

#### 3. SSL Certificate Problems
```bash
# Check certificate status
gcloud app ssl-certificates list

# Verify domain mapping
gcloud app domain-mappings describe firewallcafe.com

# Force certificate renewal
gcloud app ssl-certificates create --domains=firewallcafe.com
```

#### 4. Performance Issues
```bash
# Check instance utilization
gcloud app instances list

# Monitor resource usage
gcloud logging read "resource.type=gae_app" --limit=50

# Adjust scaling settings
gcloud app deploy --quiet  # Redeploy with updated app.yaml
```

### Debug Commands
```bash
# Stream all logs
gcloud app logs tail -s default

# Filter error logs
gcloud logging read "severity>=ERROR" --limit=50

# Check service health
curl -I https://firewallcafe.com/api/dashboard

# Database query performance
gcloud sql operations list --instance=firewall-db
```

## Maintenance

### Regular Tasks

#### Weekly
- Check error rates and performance metrics
- Review security logs
- Verify backup completion
- Update dependencies if needed

#### Monthly  
- Review and rotate API keys
- Audit IAM permissions
- Update SSL certificates (if manual)
- Performance optimization review

#### Quarterly
- Security audit and penetration testing
- Disaster recovery testing
- Cost optimization review
- Documentation updates

### Scaling Considerations

#### Horizontal Scaling
```yaml
# app.yaml - Adjust for traffic
automatic_scaling:
  min_instances: 2      # Increase for high availability
  max_instances: 50     # Increase for traffic spikes
  target_cpu_utilization: 0.6
  target_throughput_utilization: 0.6
```

#### Database Scaling
```bash
# Increase CPU/Memory
gcloud sql instances patch firewall-db \
  --cpu=4 --memory=8GB

# Add read replicas
gcloud sql instances create firewall-db-replica \
  --master-instance-name=firewall-db \
  --region=us-east1
```

## Support

### Monitoring Dashboards
- **App Engine**: https://console.cloud.google.com/appengine
- **Cloud SQL**: https://console.cloud.google.com/sql
- **Monitoring**: https://console.cloud.google.com/monitoring
- **Logging**: https://console.cloud.google.com/logs

### Emergency Contacts
- **GCP Support**: Available through console for billing account holders
- **Internal Team**: info@firewallcafe.com
- **On-call**: [Define internal escalation procedures]

---

*For additional deployment environments and advanced configurations, see [Environment Configuration Guide](./ENVIRONMENT.md)*