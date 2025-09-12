# Backup and Recovery Guide

This guide covers comprehensive backup strategies, disaster recovery procedures, and data protection measures for Firewall Cafe.

## Table of Contents
- [Backup Strategy](#backup-strategy)
- [Database Backups](#database-backups)
- [Application Code Backups](#application-code-backups)
- [Configuration Backups](#configuration-backups)
- [Automated Backup Setup](#automated-backup-setup)
- [Recovery Procedures](#recovery-procedures)
- [Disaster Recovery Planning](#disaster-recovery-planning)
- [Testing and Validation](#testing-and-validation)
- [Monitoring and Alerting](#monitoring-and-alerting)

## Backup Strategy

### Backup Types

#### 1. Database Backups
- **Primary**: Cloud SQL automatic backups (daily)
- **Secondary**: Manual exports for major releases
- **Archive**: Long-term retention for compliance

#### 2. Application Backups
- **Code**: Git repository (primary source of truth)
- **Build Artifacts**: Docker images and compiled assets
- **Configuration**: Environment variables and secrets

#### 3. External Dependencies
- **API Keys**: Secure storage in Secret Manager
- **Third-party Data**: Search result archives
- **User Data**: Analytics and voting history

### Backup Schedule

```
Daily:    Database automatic backups
Weekly:   Full database export + verification
Monthly:  Long-term archive creation
Release:  Pre-deployment snapshot
```

### Recovery Time Objectives (RTO)
- **Critical System**: 15 minutes
- **Database**: 30 minutes
- **Full Application**: 2 hours
- **Complete Disaster**: 24 hours

### Recovery Point Objectives (RPO)
- **Database**: 1 hour (point-in-time recovery)
- **Application**: Latest git commit
- **Configuration**: Latest deployment

## Database Backups

### Cloud SQL Automatic Backups

#### Enable Automatic Backups
```bash
# Configure automatic backups
gcloud sql instances patch firewall-db \
  --backup-start-time=02:00 \
  --backup-location=us \
  --enable-bin-log \
  --retained-backups-count=30 \
  --retained-transaction-log-days=7
```

#### Backup Configuration
```bash
# Set backup window (2:00 AM UTC)
gcloud sql instances patch firewall-db \
  --backup-start-time=02:00

# Enable point-in-time recovery
gcloud sql instances patch firewall-db \
  --enable-bin-log \
  --retained-transaction-log-days=7

# Configure backup retention
gcloud sql instances patch firewall-db \
  --retained-backups-count=30  # Keep 30 days of backups
```

### Manual Database Exports

#### Full Database Export
```bash
# Export entire database
EXPORT_FILE="firewall_backup_$(date +%Y%m%d_%H%M%S).sql"

gcloud sql export sql firewall-db gs://firewall-backups/$EXPORT_FILE \
  --database=firewallcafe

# Compressed export (recommended)
gcloud sql export sql firewall-db gs://firewall-backups/$EXPORT_FILE.gz \
  --database=firewallcafe \
  --offload
```

#### Table-Specific Exports
```bash
# Export specific tables
gcloud sql export sql firewall-db gs://firewall-backups/searches_backup.sql \
  --database=firewallcafe \
  --table=searches,images,votes,have_votes

# Export with custom query
gcloud sql export sql firewall-db gs://firewall-backups/recent_data.sql \
  --database=firewallcafe \
  --query="SELECT * FROM searches WHERE search_timestamp > EXTRACT(EPOCH FROM NOW() - INTERVAL '30 days') * 1000"
```

### Local Database Backups (Development)
```bash
# Create local backup
pg_dump -h localhost -U firewallcafe firewallcafe > "local_backup_$(date +%Y%m%d_%H%M%S).sql"

# Compressed backup
pg_dump -h localhost -U firewallcafe firewallcafe | gzip > "local_backup_$(date +%Y%m%d_%H%M%S).sql.gz"

# Custom format (faster restore)
pg_dump -h localhost -U firewallcafe -Fc firewallcafe > "local_backup_$(date +%Y%m%d_%H%M%S).dump"
```

### Backup Verification
```bash
# List available backups
gcloud sql backups list --instance=firewall-db

# Describe specific backup
gcloud sql backups describe BACKUP_ID --instance=firewall-db

# Test backup integrity
gcloud sql export sql firewall-db gs://firewall-backups/test_export.sql \
  --database=firewallcafe
```

## Application Code Backups

### Git Repository Management

#### Repository Backup Strategy
```bash
# Mirror repository to secondary location
git clone --mirror https://github.com/FIREWALL-cafe/firewall-cafe.git firewall-cafe-mirror.git

# Update mirror regularly
cd firewall-cafe-mirror.git
git remote update

# Create periodic archives
git archive --format=tar.gz --prefix=firewall-cafe/ HEAD > "firewall-cafe-$(date +%Y%m%d).tar.gz"
```

#### Branch Protection
```bash
# Configure branch protection via GitHub API
curl -X PUT \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/FIREWALL-cafe/firewall-cafe/branches/main/protection \
  -d '{
    "required_status_checks": {
      "strict": true,
      "contexts": ["continuous-integration/github-actions"]
    },
    "enforce_admins": false,
    "required_pull_request_reviews": {
      "required_approving_review_count": 1
    },
    "restrictions": null
  }'
```

### Build Artifacts Backup

#### Container Images
```bash
# Tag and push images to multiple registries
docker tag firewall-cafe:latest gcr.io/firewall-cafe-prod/firewall-cafe:$(date +%Y%m%d)
docker push gcr.io/firewall-cafe-prod/firewall-cafe:$(date +%Y%m%d)

# Backup to secondary registry
docker tag firewall-cafe:latest backup-registry.com/firewall-cafe:$(date +%Y%m%d)
docker push backup-registry.com/firewall-cafe:$(date +%Y%m%d)
```

#### Static Assets
```bash
# Backup built assets to Cloud Storage
gsutil -m cp -r build/ gs://firewall-backups/assets/$(date +%Y%m%d)/

# Create versioned backup
tar -czf "assets-backup-$(date +%Y%m%d).tar.gz" build/
gsutil cp "assets-backup-$(date +%Y%m%d).tar.gz" gs://firewall-backups/assets/
```

## Configuration Backups

### Environment Variables Backup
```bash
# Export current environment configuration (excluding secrets)
cat > environment_backup.env << EOF
# Environment backup created $(date)
NODE_ENV=${NODE_ENV}
DB_HOST=${DB_HOST}
DB_PORT=${DB_PORT}
DB_USER=${DB_USER}
DB_NAME=${DB_NAME}
# Secrets omitted for security
EOF

# Store in secure location
gsutil cp environment_backup.env gs://firewall-backups/config/$(date +%Y%m%d)/
```

### Secret Manager Backup
```bash
# List all secrets
gcloud secrets list --format="value(name)" > secrets_list.txt

# Export secret metadata (not values)
while read -r secret_name; do
  gcloud secrets describe "$secret_name" --format=yaml > "secret_${secret_name}.yaml"
done < secrets_list.txt

# Archive secret configurations
tar -czf "secrets_config_$(date +%Y%m%d).tar.gz" secret_*.yaml
gsutil cp "secrets_config_$(date +%Y%m%d).tar.gz" gs://firewall-backups/secrets/
```

### Application Configuration
```bash
# Backup App Engine configuration
gsutil cp app.yaml gs://firewall-backups/config/$(date +%Y%m%d)/app.yaml

# Backup Cloud Build configuration
gsutil cp cloudbuild.yaml gs://firewall-backups/config/$(date +%Y%m%d)/cloudbuild.yaml

# Backup monitoring configuration
gcloud alpha monitoring policies list --format=yaml > monitoring_policies.yaml
gsutil cp monitoring_policies.yaml gs://firewall-backups/config/$(date +%Y%m%d)/
```

## Automated Backup Setup

### Cloud Scheduler for Database Backups
```bash
# Create Cloud Scheduler job for weekly full backup
gcloud scheduler jobs create app-engine weekly-db-backup \
  --schedule="0 2 * * 0" \
  --timezone="UTC" \
  --uri="/admin/backup" \
  --http-method=POST \
  --headers="Content-Type=application/json" \
  --message-body='{"type":"full","notify":true}'
```

### Cloud Function for Automated Backups
```javascript
// functions/backup/index.js
const { Storage } = require('@google-cloud/storage');
const { execSync } = require('child_process');

exports.performBackup = async (req, res) => {
  const backupType = req.body.type || 'database';
  const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
  
  try {
    switch (backupType) {
      case 'database':
        await backupDatabase(timestamp);
        break;
      case 'full':
        await performFullBackup(timestamp);
        break;
      default:
        throw new Error(`Unknown backup type: ${backupType}`);
    }
    
    res.status(200).json({
      success: true,
      timestamp,
      type: backupType
    });
    
  } catch (error) {
    console.error('Backup failed:', error);
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

async function backupDatabase(timestamp) {
  const backupName = `firewall_db_${timestamp}.sql`;
  
  // Export database
  const exportCommand = `gcloud sql export sql firewall-db gs://firewall-backups/database/${backupName} --database=firewallcafe`;
  execSync(exportCommand);
  
  // Verify backup
  const storage = new Storage();
  const file = storage.bucket('firewall-backups').file(`database/${backupName}`);
  const [exists] = await file.exists();
  
  if (!exists) {
    throw new Error('Backup file not found after export');
  }
  
  console.log(`Database backup completed: ${backupName}`);
  return backupName;
}

async function performFullBackup(timestamp) {
  const tasks = [];
  
  // Database backup
  tasks.push(backupDatabase(timestamp));
  
  // Configuration backup
  tasks.push(backupConfiguration(timestamp));
  
  // Wait for all backups to complete
  await Promise.all(tasks);
  
  console.log(`Full backup completed: ${timestamp}`);
}

async function backupConfiguration(timestamp) {
  // Export current configurations
  const configs = [
    'app.yaml',
    'cloudbuild.yaml',
    'package.json'
  ];
  
  for (const config of configs) {
    const backupPath = `config/${timestamp}/${config}`;
    const copyCommand = `gsutil cp ${config} gs://firewall-backups/${backupPath}`;
    execSync(copyCommand);
  }
}
```

### Backup Monitoring Script
```bash
#!/bin/bash
# backup-monitor.sh - Monitor backup health

BACKUP_BUCKET="gs://firewall-backups"
SLACK_WEBHOOK_URL="YOUR_SLACK_WEBHOOK"

# Check if daily backup exists
TODAY=$(date +%Y%m%d)
BACKUP_EXISTS=$(gsutil ls "${BACKUP_BUCKET}/database/" | grep "$TODAY" | wc -l)

if [ "$BACKUP_EXISTS" -eq 0 ]; then
  # Alert missing backup
  curl -X POST "$SLACK_WEBHOOK_URL" \
    -H 'Content-type: application/json' \
    --data '{"text":"⚠️ Missing daily database backup for '"$TODAY"'"}'
  exit 1
fi

# Check backup size (should be > 1MB for our database)
LATEST_BACKUP=$(gsutil ls -l "${BACKUP_BUCKET}/database/" | grep "$TODAY" | tail -1)
BACKUP_SIZE=$(echo "$LATEST_BACKUP" | awk '{print $1}')

if [ "$BACKUP_SIZE" -lt 1048576 ]; then
  curl -X POST "$SLACK_WEBHOOK_URL" \
    -H 'Content-type: application/json' \
    --data '{"text":"⚠️ Database backup appears to be too small: '"$BACKUP_SIZE"' bytes"}'
  exit 1
fi

echo "✅ Backup verification passed for $TODAY"
```

## Recovery Procedures

### Database Recovery

#### Point-in-Time Recovery
```bash
# List available backups
gcloud sql backups list --instance=firewall-db

# Restore to specific point in time
gcloud sql backups restore BACKUP_ID \
  --restore-instance=firewall-db \
  --backup-instance=firewall-db

# Create new instance from backup
gcloud sql instances clone firewall-db firewall-db-restored \
  --backup-id=BACKUP_ID
```

#### Recovery from Export
```bash
# Create temporary instance for recovery
gcloud sql instances create firewall-db-temp \
  --database-version=POSTGRES_14 \
  --cpu=2 \
  --memory=4GB \
  --region=us-central1

# Import from backup file
gcloud sql import sql firewall-db-temp gs://firewall-backups/firewall_backup_TIMESTAMP.sql \
  --database=firewallcafe

# Verify data integrity
gcloud sql connect firewall-db-temp --user=postgres
# Run verification queries
```

#### Local Database Recovery
```bash
# Drop and recreate database
dropdb -U postgres firewallcafe
createdb -U postgres firewallcafe

# Restore from SQL dump
psql -U postgres -d firewallcafe < backup_file.sql

# Restore from custom format
pg_restore -U postgres -d firewallcafe backup_file.dump

# Verify restoration
psql -U postgres -d firewallcafe -c "SELECT COUNT(*) FROM searches;"
```

### Application Recovery

#### Rollback to Previous Version
```bash
# List App Engine versions
gcloud app versions list

# Split traffic to previous version
gcloud app services set-traffic default --splits=PREVIOUS_VERSION=100

# Delete problematic version (after verification)
gcloud app versions delete CURRENT_VERSION
```

#### Recovery from Git
```bash
# Identify last known good commit
git log --oneline -10

# Create recovery branch
git checkout -b recovery/rollback-to-COMMIT_HASH COMMIT_HASH

# Deploy recovery version
yarn build
gcloud app deploy --version=recovery --no-promote

# Test recovery version
curl -I https://recovery-dot-firewall-cafe-prod.ue.r.appspot.com/health

# Promote if successful
gcloud app services set-traffic default --splits=recovery=100
```

### Configuration Recovery

#### Restore Environment Variables
```bash
# Download backup configuration
gsutil cp gs://firewall-backups/config/BACKUP_DATE/environment_backup.env .

# Update App Engine configuration
gcloud app deploy --env-vars-file=environment_backup.env
```

#### Restore Secrets
```bash
# List backed up secrets
gsutil ls gs://firewall-backups/secrets/

# Download secret configurations
gsutil cp gs://firewall-backups/secrets/secrets_config_TIMESTAMP.tar.gz .
tar -xzf secrets_config_TIMESTAMP.tar.gz

# Recreate secrets (values must be entered manually)
gcloud secrets create secret-name --data-file=-
# Enter secret value when prompted
```

## Disaster Recovery Planning

### Multi-Region Setup

#### Primary Region: us-central1
- Production database and application
- Automatic backups enabled
- Real-time monitoring

#### Secondary Region: us-east1
- Read replica database
- Standby App Engine service
- Cross-region backup replication

```bash
# Create read replica
gcloud sql instances create firewall-db-replica \
  --master-instance-name=firewall-db \
  --region=us-east1 \
  --replica-type=READ

# Deploy to secondary region
gcloud app deploy --project=firewall-cafe-dr --version=standby
```

### Failover Procedures

#### Automatic Failover Setup
```bash
# Configure load balancer for automatic failover
gcloud compute url-maps create firewall-lb \
  --default-service=primary-backend-service

# Add health check
gcloud compute health-checks create http firewall-health-check \
  --port=8080 \
  --request-path=/health

# Configure backend services
gcloud compute backend-services create primary-backend-service \
  --protocol=HTTP \
  --health-checks=firewall-health-check \
  --global
```

#### Manual Failover Process
```bash
# 1. Promote read replica to master
gcloud sql instances promote-replica firewall-db-replica

# 2. Update application configuration
gcloud app deploy --env-vars-file=failover-config.env

# 3. Switch DNS to backup region
# Update DNS records to point to us-east1 deployment

# 4. Notify stakeholders
curl -X POST $SLACK_WEBHOOK_URL \
  -d '{"text":"🚨 Failover to disaster recovery region activated"}'
```

### Recovery Time Estimation

| Scenario | Recovery Time | Steps |
|----------|---------------|--------|
| App Engine rollback | 5-10 minutes | Version switch via traffic split |
| Database point-in-time recovery | 15-30 minutes | Cloud SQL restore operation |
| Full region failover | 30-60 minutes | Replica promotion + DNS changes |
| Complete disaster recovery | 2-4 hours | New infrastructure + data recovery |

## Testing and Validation

### Backup Testing Schedule

#### Monthly Testing
```bash
#!/bin/bash
# monthly-backup-test.sh

set -e

echo "🧪 Starting monthly backup test..."

# 1. Create test database from latest backup
LATEST_BACKUP=$(gcloud sql backups list --instance=firewall-db --limit=1 --format="value(id)")
gcloud sql instances clone firewall-db test-restore-$(date +%Y%m%d) \
  --backup-id=$LATEST_BACKUP

# 2. Verify data integrity
TEST_INSTANCE="test-restore-$(date +%Y%m%d)"
SEARCH_COUNT=$(gcloud sql execute-sql $TEST_INSTANCE --sql="SELECT COUNT(*) FROM searches;" --format="value(result[0].rows[0][0])")

if [ "$SEARCH_COUNT" -gt 1000 ]; then
  echo "✅ Backup test passed: $SEARCH_COUNT searches found"
else
  echo "❌ Backup test failed: Only $SEARCH_COUNT searches found"
  exit 1
fi

# 3. Clean up test instance
gcloud sql instances delete $TEST_INSTANCE --quiet

echo "🎉 Monthly backup test completed successfully"
```

#### Quarterly Disaster Recovery Drill
```bash
#!/bin/bash
# quarterly-dr-drill.sh

echo "🚨 Starting disaster recovery drill..."

# 1. Deploy to DR region
gcloud app deploy --project=firewall-cafe-dr --version=dr-drill-$(date +%Y%m%d)

# 2. Test application functionality
DR_URL="https://dr-drill-$(date +%Y%m%d)-dot-firewall-cafe-dr.ue.r.appspot.com"

# Test health endpoint
if curl -f "$DR_URL/health" > /dev/null 2>&1; then
  echo "✅ DR application health check passed"
else
  echo "❌ DR application health check failed"
  exit 1
fi

# Test search functionality
if curl -f "$DR_URL/api/searches?limit=1" > /dev/null 2>&1; then
  echo "✅ DR search API test passed"
else
  echo "❌ DR search API test failed"
  exit 1
fi

echo "🎉 Disaster recovery drill completed successfully"
```

### Validation Scripts

#### Database Integrity Check
```sql
-- database-integrity-check.sql
-- Run after backup restoration to verify data integrity

-- Check for orphaned records
SELECT 'Orphaned images' as check_type, COUNT(*) as count
FROM images i
LEFT JOIN searches s ON i.search_id = s.search_id
WHERE s.search_id IS NULL;

-- Check vote referential integrity
SELECT 'Invalid votes' as check_type, COUNT(*) as count
FROM have_votes hv
LEFT JOIN searches s ON hv.search_id = s.search_id
LEFT JOIN votes v ON hv.vote_id = v.vote_id
WHERE s.search_id IS NULL OR v.vote_id IS NULL;

-- Check for data consistency
SELECT 
  'Recent data' as check_type,
  COUNT(*) as count,
  MAX(search_timestamp) as latest_timestamp
FROM searches
WHERE search_timestamp > EXTRACT(EPOCH FROM NOW() - INTERVAL '24 hours') * 1000;

-- Verify table sizes
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size,
  pg_stat_get_tuples_returned(c.oid) as row_count
FROM pg_tables pt
JOIN pg_class c ON c.relname = pt.tablename
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## Monitoring and Alerting

### Backup Monitoring
```javascript
// monitoring/backupMonitor.js
const { Storage } = require('@google-cloud/storage');
const { SlackAlerts } = require('./slack');

class BackupMonitor {
  constructor() {
    this.storage = new Storage();
    this.backupBucket = 'firewall-backups';
    this.slack = new SlackAlerts(process.env.SLACK_WEBHOOK_URL);
  }
  
  async checkDailyBackup() {
    const today = new Date().toISOString().split('T')[0].replace(/-/g, '');
    
    try {
      const [files] = await this.storage
        .bucket(this.backupBucket)
        .getFiles({ prefix: `database/firewall_db_${today}` });
        
      if (files.length === 0) {
        await this.slack.sendAlert(
          'critical',
          'Missing Daily Backup',
          `No database backup found for ${today}`
        );
        return false;
      }
      
      // Check backup size
      const latestBackup = files[files.length - 1];
      const [metadata] = await latestBackup.getMetadata();
      const sizeGB = parseInt(metadata.size) / (1024 * 1024 * 1024);
      
      if (sizeGB < 0.1) {
        await this.slack.sendAlert(
          'warning',
          'Small Backup File',
          `Backup file appears unusually small: ${sizeGB.toFixed(2)}GB`
        );
      }
      
      return true;
      
    } catch (error) {
      await this.slack.sendAlert(
        'critical',
        'Backup Check Failed',
        `Error checking daily backup: ${error.message}`
      );
      return false;
    }
  }
  
  async checkBackupRetention() {
    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
    
    try {
      const [files] = await this.storage
        .bucket(this.backupBucket)
        .getFiles({ prefix: 'database/' });
        
      const oldBackups = files.filter(file => {
        const [metadata] = file.metadata;
        return new Date(metadata.timeCreated) < thirtyDaysAgo;
      });
      
      if (oldBackups.length > 35) { // Should keep ~30 days + buffer
        await this.slack.sendAlert(
          'info',
          'Backup Cleanup Needed',
          `${oldBackups.length} old backups found, cleanup recommended`
        );
      }
      
    } catch (error) {
      console.error('Error checking backup retention:', error);
    }
  }
}

// Scheduled monitoring function
exports.monitorBackups = async (req, res) => {
  const monitor = new BackupMonitor();
  
  const dailyBackupOk = await monitor.checkDailyBackup();
  await monitor.checkBackupRetention();
  
  res.json({
    success: true,
    dailyBackupPresent: dailyBackupOk,
    timestamp: new Date().toISOString()
  });
};
```

### Backup Alerting Rules
```yaml
# Cloud Monitoring alert for backup failures
displayName: "Backup Failure Alert"
conditions:
  - displayName: "Cloud SQL backup failed"
    conditionThreshold:
      filter: 'resource.type="cloudsql_database" AND log_name="projects/PROJECT_ID/logs/cloudsql.googleapis.com%2Fpostgres.log" AND textPayload:"backup failed"'
      comparison: COMPARISON_GREATER_THAN
      thresholdValue: 0
      duration: "60s"
      
alertStrategy:
  autoClose: "3600s"
enabled: true
```

### Recovery Testing Alerts
```bash
# Create Cloud Scheduler job for backup testing
gcloud scheduler jobs create app-engine monthly-backup-test \
  --schedule="0 3 1 * *" \
  --timezone="UTC" \
  --uri="/admin/test-backup" \
  --http-method=POST
```

---

*This backup and recovery guide should be reviewed and tested regularly. Update procedures as the system evolves and verify all recovery processes during planned maintenance windows.*