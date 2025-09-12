# Vercel Migration Guide

A comprehensive guide for migrating Firewall Cafe from Google Cloud Platform (App Engine) to Vercel.

## Table of Contents
- [Overview](#overview)
- [Migration Benefits](#migration-benefits)
- [Pre-Migration Checklist](#pre-migration-checklist)
- [Migration Steps](#migration-steps)
- [Configuration Changes](#configuration-changes)
- [Database Considerations](#database-considerations)
- [API & Backend Services](#api--backend-services)
- [Deployment Scripts](#deployment-scripts)
- [Testing & Validation](#testing--validation)
- [Rollback Plan](#rollback-plan)
- [Post-Migration Tasks](#post-migration-tasks)

## Overview

This guide covers migrating the Firewall Cafe client application from Google Cloud Platform's App Engine to Vercel's edge network platform. The migration focuses on the React frontend application while maintaining the existing PostgreSQL database infrastructure.

### Current Architecture
- **Frontend**: React app on Google App Engine
- **Backend API**: Node.js/Express on App Engine (port 11458)
- **Proxy Server**: Express proxy server (port 8080)
- **Database**: PostgreSQL on Cloud SQL
- **Build System**: Create React App with custom scripts

### Target Architecture
- **Frontend**: React app on Vercel Edge Network
- **Backend API**: Options:
  - Keep on GCP App Engine (hybrid approach)
  - Migrate to Vercel Functions
  - Use separate backend service (Railway, Render, etc.)
- **Database**: Keep PostgreSQL on Cloud SQL or migrate to Vercel Postgres/Supabase

## Migration Benefits

### Performance Improvements
- **Edge Network**: Deploy to 100+ global edge locations
- **Automatic CDN**: Built-in CDN with smart caching
- **Instant Cache Invalidation**: Deploy changes instantly
- **Optimized Build Output**: Automatic optimizations for React apps

### Developer Experience
- **Git Integration**: Deploy on push to GitHub
- **Preview Deployments**: Automatic preview URLs for PRs
- **Instant Rollbacks**: One-click rollbacks to previous versions
- **Zero Config**: Works out-of-box with Create React App

### Cost Optimization
- **Free Tier**: Generous free tier for small projects
- **Pay-per-Use**: Only pay for what you use
- **No Minimum Instances**: Unlike App Engine's min_instances
- **Serverless Functions**: Pay only when functions execute

## Pre-Migration Checklist

### Prerequisites
- [ ] Vercel account created
- [ ] GitHub repository connected to Vercel
- [ ] Environment variables documented
- [ ] Database connection strings ready
- [ ] API keys and secrets documented
- [ ] Custom domain DNS access
- [ ] Backup of current production data

### Inventory Current Setup
- [ ] List all environment variables in use
- [ ] Document all API endpoints
- [ ] Note custom headers and redirects
- [ ] Identify static assets and their paths
- [ ] Review current scaling settings
- [ ] Document current deployment process

## Migration Steps

### Step 1: Prepare Project Structure

1. **Fix Configuration Imports**
   
   Since Vercel builds the client separately, ensure all client components import from client-side config files:
   
   - Move any shared config values to `client/src/config.js`
   - Update component imports from `../config` to use the local config
   - Use environment variables for sensitive values
   
   **Important**: The `client/server/config.js` file is for the Express proxy server and won't be accessible during the React build process.

2. **Create Vercel Configuration**
   
   Place this `vercel.json` file in the `client/` directory (not the root):
   
   ```json
   // client/vercel.json
   {
     "framework": "create-react-app",
     "buildCommand": "yarn build",
     "outputDirectory": "build",
     "devCommand": "yarn start:client",
     "installCommand": "yarn install",
     "regions": ["iad1"],
     "functions": {
       "api/proxy.js": {
         "maxDuration": 30
       }
     },
     "rewrites": [
       {
         "source": "/api/:path*",
         "destination": "/api/proxy"
       },
       {
         "source": "/proxy-image",
         "destination": "/api/proxy-image"
       }
     ],
     "headers": [
       {
         "source": "/static/(.*)",
         "headers": [
           {
             "key": "Cache-Control",
             "value": "public, max-age=31536000, immutable"
           }
         ]
       }
     ],
     "env": {
       "REACT_APP_API_URL": "@api-url",
       "REACT_APP_BACKEND_API_URL": "@backend-api-url"
     }
   }
   ```

2. **Update Package.json Scripts**
   ```json
   {
     "scripts": {
       "predeploy": "yarn run format && yarn run lint && yarn run build",
       "deploy:vercel": "vercel --prod",
       "deploy:preview": "vercel",
       "dev": "vercel dev"
     }
   }
   ```

### Step 2: Convert Proxy Server to Vercel Functions

1. **Create API Directory Structure**
   ```
   client/
   ├── api/
   │   ├── proxy.js          # Main proxy handler
   │   ├── proxy-image.js    # Image proxy handler
   │   └── translate.js      # Translation API handler
   ```

2. **Convert Express Proxy to Vercel Function**
   ```javascript
   // api/proxy.js
   import fetch from 'node-fetch';

   export default async function handler(req, res) {
     const { path } = req.query;
     const backendUrl = process.env.BACKEND_API_URL || 'http://localhost:11458';
     
     try {
       const response = await fetch(`${backendUrl}/api/${path}`, {
         method: req.method,
         headers: {
           ...req.headers,
           host: undefined,
         },
         body: req.method !== 'GET' ? JSON.stringify(req.body) : undefined,
       });

       const data = await response.json();
       res.status(response.status).json(data);
     } catch (error) {
       res.status(500).json({ error: 'Proxy error', details: error.message });
     }
   }

   export const config = {
     api: {
       bodyParser: {
         sizeLimit: '10mb',
       },
     },
   };
   ```

3. **Image Proxy Function**
   ```javascript
   // api/proxy-image.js
   import fetch from 'node-fetch';

   export default async function handler(req, res) {
     const { url } = req.query;
     
     if (!url) {
       return res.status(400).json({ error: 'URL parameter required' });
     }

     try {
       const response = await fetch(url);
       const buffer = await response.buffer();
       const contentType = response.headers.get('content-type');
       
       res.setHeader('Content-Type', contentType);
       res.setHeader('Cache-Control', 'public, max-age=86400');
       res.send(buffer);
     } catch (error) {
       res.status(500).json({ error: 'Failed to proxy image' });
     }
   }
   ```

### Step 3: Environment Variables Setup

1. **Install Vercel CLI**
   ```bash
   npm i -g vercel
   ```

2. **Set Environment Variables**
   ```bash
   # Production secrets
   vercel env add BACKEND_API_URL production
   vercel env add SERPER_API_KEY production
   vercel env add GOOGLE_TRANSLATE_API_KEY production
   vercel env add SHARED_SECRET production

   # Database connection (if using Vercel Functions for backend)
   vercel env add DATABASE_URL production
   
   # Preview environment
   vercel env add BACKEND_API_URL preview
   ```

### Step 4: Database Migration Options

#### Option A: Keep Cloud SQL (Recommended for Phase 1)
- Maintain existing Cloud SQL instance
- Update connection strings for Vercel Functions
- Use Cloud SQL Proxy for secure connections

#### Option B: Migrate to Vercel Postgres
```bash
# Create Vercel Postgres database
vercel postgres create firewall-db

# Get connection string
vercel env pull .env.local

# Migrate data using pg_dump
pg_dump postgresql://old-connection | psql $POSTGRES_URL
```

#### Option C: Use Supabase
- Create Supabase project
- Migrate data using Supabase migration tools
- Update connection strings

### Step 5: Deploy to Vercel

1. **Configure for Subdirectory Deployment**
   
   Since the React app is in the `client/` subdirectory, configure Vercel accordingly:
   
   **Option A: Via Vercel Dashboard (Recommended)**
   - Import your Git repository
   - Set "Root Directory" to `client` (this is set in the dashboard, not in vercel.json)
   - Framework Preset: Create React App
   - Build Command: `yarn build`
   - Output Directory: `build`
   
   **Note**: The `root` property is not valid in `vercel.json`. The root directory must be configured in the Vercel dashboard project settings or via CLI during project setup.
   
   **Option B: Via CLI from client directory**
   ```bash
   cd client
   vercel login
   vercel link
   vercel --prod
   ```

2. **Initial Deployment**
   ```bash
   # From the client directory
   cd client
   
   # Login to Vercel
   vercel login

   # Link project (follow prompts)
   vercel link

   # Deploy to preview
   vercel

   # Deploy to production
   vercel --prod
   ```

3. **Connect GitHub Repository**
   ```bash
   # In Vercel Dashboard:
   # 1. Import Git Repository
   # 2. Select FIREWALL-cafe/firewall-cafe
   # 3. Set Root Directory to "client"
   # 4. Configure build settings
   # 5. Set environment variables
   # 6. Deploy
   ```

### Step 6: Update DNS Records

1. **Get Vercel DNS Settings**
   ```bash
   vercel domains add firewallcafe.com
   ```

2. **Update DNS Records**
   ```
   # Remove old GCP records
   # Add Vercel records
   A     firewallcafe.com → 76.76.21.21
   CNAME www → cname.vercel-dns.com
   ```

## Configuration Changes

### Build Configuration
```javascript
// next.config.js (if migrating to Next.js)
module.exports = {
  reactStrictMode: true,
  images: {
    domains: ['firewallcafe.com'],
  },
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'http://localhost:11458/api/:path*',
      },
    ];
  },
};
```

### Headers and Redirects
```json
// vercel.json
{
  "redirects": [
    {
      "source": "/old-path",
      "destination": "/new-path",
      "permanent": true
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        }
      ]
    }
  ]
}
```

## API & Backend Services

### Hybrid Approach (Recommended)
Keep backend API on GCP while moving frontend to Vercel:

1. **Update CORS Settings**
   ```javascript
   // server/api/index.js
   app.use(cors({
     origin: [
       'https://firewallcafe.com',
       'https://firewall-cafe.vercel.app',
       /\.vercel\.app$/
     ],
     credentials: true
   }));
   ```

2. **Update API URLs**
   ```javascript
   // client/src/config.js
   export const API_URL = process.env.REACT_APP_API_URL || 
     'https://api.firewallcafe.com';
   ```

### Full Migration to Vercel Functions
Convert Express routes to Vercel Functions:

```javascript
// api/searches.js
export default async function handler(req, res) {
  const { method } = req;
  
  switch (method) {
    case 'GET':
      // Handle GET request
      break;
    case 'POST':
      // Handle POST request
      break;
    default:
      res.setHeader('Allow', ['GET', 'POST']);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}
```

## Deployment Scripts

### Update Deploy Script
```javascript
// scripts/deploy-vercel.js
#!/usr/bin/env node

const { execSync } = require('child_process');

function deploy(environment = 'production') {
  console.log(`🚀 Deploying to Vercel (${environment})...`);
  
  // Run pre-deploy checks
  execSync('yarn run predeploy', { stdio: 'inherit' });
  
  // Deploy to Vercel
  const command = environment === 'production' 
    ? 'vercel --prod' 
    : 'vercel';
  
  execSync(command, { stdio: 'inherit' });
  
  console.log('✅ Deployment complete!');
}

// Get environment from command line
const env = process.argv[2] || 'production';
deploy(env);
```

### Rollback Script for Vercel
```javascript
// scripts/rollback-vercel.js
#!/usr/bin/env node

const { execSync } = require('child_process');
const readline = require('readline');

async function rollback() {
  // List recent deployments
  console.log('📋 Recent deployments:');
  const deployments = execSync('vercel ls --json', { encoding: 'utf8' });
  const parsed = JSON.parse(deployments);
  
  parsed.deployments.slice(0, 5).forEach((d, i) => {
    console.log(`${i + 1}. ${d.url} - ${d.created}`);
  });
  
  // Prompt for selection
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
  });
  
  rl.question('Select deployment to rollback to: ', (answer) => {
    const index = parseInt(answer) - 1;
    const deployment = parsed.deployments[index];
    
    // Rollback using alias
    execSync(`vercel alias ${deployment.url} firewallcafe.com`, 
      { stdio: 'inherit' });
    
    console.log('✅ Rollback complete!');
    rl.close();
  });
}

rollback();
```

## Testing & Validation

### Pre-Deployment Testing
```bash
# Local testing with Vercel CLI
vercel dev

# Build verification
yarn build
npx serve -s build

# Environment variable check
vercel env ls
```

### Post-Deployment Validation
- [ ] Homepage loads correctly
- [ ] Search functionality works
- [ ] API endpoints respond
- [ ] Images load through proxy
- [ ] Geographic features work
- [ ] Vote system functions
- [ ] Archive search works
- [ ] Dashboard displays data

### Performance Testing
```bash
# Lighthouse CI
npm install -g @lhci/cli
lhci autorun

# WebPageTest
curl -X POST https://www.webpagetest.org/runtest.php
```

## Rollback Plan

### Immediate Rollback
1. **Revert DNS**: Point domain back to GCP
2. **Restore App Engine**: Deploy previous version
   ```bash
   gcloud app versions list
   gcloud app services set-traffic default --splits=v1=1
   ```

### Gradual Rollback
1. **Traffic Splitting**: Use Vercel's traffic splitting
2. **Monitor Metrics**: Watch error rates
3. **Adjust Traffic**: Gradually move traffic back

## Post-Migration Tasks

### Week 1
- [ ] Monitor error rates and performance
- [ ] Collect user feedback
- [ ] Fine-tune caching rules
- [ ] Optimize function cold starts

### Week 2
- [ ] Review costs comparison
- [ ] Document new workflows
- [ ] Update CI/CD pipelines
- [ ] Train team on Vercel dashboard

### Month 1
- [ ] Full performance audit
- [ ] Security review
- [ ] Cost optimization
- [ ] Consider full backend migration

## Troubleshooting

### Common Issues

#### Config Import Error
**Error**: `Module not found: Error: Can't resolve '../config' in '/vercel/path0/src/components'`

**Cause**: Module format mismatch or missing config file. The config file uses CommonJS exports but is being imported with ES6 syntax.

**Solution Options**:

**Option 1: Convert to ES6 Modules (Recommended)**
Update `client/src/config.js` to use ES6 export syntax:
```javascript
// client/src/config.js
const config = {
  displayVoting: true,
  proxyImages: true,
};

export default config;
```

**Option 2: Keep CommonJS but ensure compatibility**
If keeping CommonJS format, ensure the file exists at `client/src/config.js`:
```javascript
// client/src/config.js
module.exports = {
  displayVoting: true,
  proxyImages: true,
};
```

And verify webpack/Create React App can handle it (it should by default).

**Option 3: Use Environment Variables (Best for Production)**
Replace config file with environment variables:
```javascript
// client/src/components/SearchCompare.jsx
const displayVoting = process.env.REACT_APP_DISPLAY_VOTING === 'true';

// Or create a config module
// client/src/config/index.js
export const displayVoting = process.env.REACT_APP_DISPLAY_VOTING !== 'false';
export const proxyImages = process.env.REACT_APP_PROXY_IMAGES !== 'false';
```

Then set in Vercel:
```bash
vercel env add REACT_APP_DISPLAY_VOTING production
vercel env add REACT_APP_PROXY_IMAGES production
```

**Debugging Steps**:
1. Verify the file exists: `ls client/src/config.js`
2. Check file is committed: `git status client/src/config.js`
3. Ensure it's not in `.gitignore`
4. Try clearing Vercel cache: Dashboard > Settings > Advanced > Clear Cache

#### Changes Not Deploying
**Problem**: Pushed changes to repository but they aren't showing in Vercel deployment

**Common Causes & Solutions**:

1. **Wrong Branch**
   - Check which branch Vercel is watching (usually `main` or `master`)
   - Verify in Vercel Dashboard > Settings > Git > Production Branch
   - Ensure you pushed to the correct branch:
     ```bash
     git branch  # Check current branch
     git push origin main  # Push to main branch
     ```

2. **Git Hook Not Triggered**
   - Check if automatic deployments are enabled: Dashboard > Settings > Git > Deploy Hooks
   - Manually trigger deployment: Dashboard > Deployments > Redeploy

3. **Ignored Build Step**
   - Vercel might skip builds if no changes detected in root directory
   - Check "Ignored Build Step" in Project Settings
   - Force rebuild: `vercel --force` or click "Redeploy" in dashboard

4. **Root Directory Mismatch**
   - Verify root directory setting matches your changes location
   - If set to `client/`, ensure changes are in that directory
   - Check: Dashboard > Settings > General > Root Directory

5. **Cache Issues**
   - Clear build cache: Dashboard > Settings > Advanced > Clear Cache
   - Or use CLI: `vercel --force`

6. **Git Integration Issues**
   - Disconnect and reconnect GitHub integration
   - Check repository permissions in GitHub settings
   - Verify webhook is active: GitHub repo > Settings > Webhooks

**Quick Diagnosis**:
```bash
# Check deployment status
vercel ls

# View recent deployment logs
vercel logs

# Manually deploy from CLI to test
vercel --prod
```

#### Vercel.json Schema Validation Error
**Error**: `The vercel.json schema validation failed: should NOT have additional property 'root'`

**Solution**: Remove the `root` property from `vercel.json`. The root directory should be configured in the Vercel dashboard under Project Settings > General > Root Directory, not in the configuration file.

#### Build Failures
```bash
# Clear cache and rebuild
vercel --force

# Check build logs
vercel logs
```

#### Environment Variables Not Working
```bash
# Pull env vars locally
vercel env pull .env.local

# Verify in dashboard
vercel env ls
```

#### Function Timeouts
```json
// vercel.json
{
  "functions": {
    "api/*.js": {
      "maxDuration": 60
    }
  }
}
```

## Cost Comparison

### Google Cloud Platform (Current)
- App Engine: ~$50-200/month
- Cloud SQL: ~$50-100/month
- Cloud Storage: ~$10/month
- **Total**: ~$110-310/month

### Vercel (Projected)
- Pro Plan: $20/month
- Vercel Postgres: $15/month (or keep Cloud SQL)
- Bandwidth: ~$20/month
- **Total**: ~$55-155/month

## Resources

### Documentation
- [Vercel Documentation](https://vercel.com/docs)
- [Create React App on Vercel](https://vercel.com/guides/deploying-react-with-vercel-cra)

### Support
- Vercel Support: support@vercel.com
- Community: https://github.com/vercel/vercel/discussions
- Status: https://vercel-status.com

---

*Last updated: 2025-09-05*
*For questions about this migration, contact the development team.*