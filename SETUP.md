# Development Setup Guide

This guide will help you set up the Firewall Cafe development environment on your local machine.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Database Setup](#database-setup)
- [API Configuration](#api-configuration)
- [Running the Application](#running-the-application)
- [Verification](#verification)
- [Common Issues](#common-issues)
- [Development Tools](#development-tools)

## Prerequisites

### Required Software

1. **Node.js v20.x**
   ```bash
   # Install nvm (Node Version Manager)
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
   
   # Install and use Node v20
   nvm install 20
   nvm use 20
   ```

2. **Yarn Package Manager**
   ```bash
   npm install -g yarn
   ```

3. **PostgreSQL 14+**
   ```bash
   # macOS
   brew install postgresql@14
   brew services start postgresql@14
   
   # Ubuntu/Debian
   sudo apt update
   sudo apt install postgresql postgresql-contrib
   sudo systemctl start postgresql
   ```

4. **Git**
   ```bash
   # Verify git is installed
   git --version
   ```

### Recommended Tools

- **VS Code** or your preferred code editor
- **Postman** or similar for API testing
- **pgAdmin** or TablePlus for database management
- **Chrome DevTools** for frontend debugging

## Environment Setup

### 1. Clone the Repository

```bash
# Clone the repository
git clone https://github.com/FIREWALL-cafe/firewall-cafe.git
cd firewall-cafe

# Or if you have SSH set up
git clone git@github.com:FIREWALL-cafe/firewall-cafe.git
cd firewall-cafe
```

### 2. Install Dependencies

```bash
# Install client dependencies
cd client
nvm use  # This will use the version specified in .nvmrc
yarn install

# Install server dependencies
cd ../server/api
npm install
```

### 3. Environment Variables

Create environment files for both client and server:

#### Client Environment (.env)
```bash
cd client
cp .env.example .env
```

Edit `client/.env`:
```env
# API Configuration
REACT_APP_API_URL=http://localhost:8080
REACT_APP_BACKEND_API_URL=http://localhost:11458

# Search APIs
REACT_APP_SERPER_API_KEY=your_serper_api_key_here
REACT_APP_GOOGLE_TRANSLATE_API_KEY=your_translate_key_here

# Feature Flags
REACT_APP_ENABLE_VOTING=true
REACT_APP_ENABLE_ANALYTICS=true
REACT_APP_ENABLE_HEATMAPS=true

# Development
REACT_APP_DEBUG_MODE=true
```

#### Server Configuration
```bash
cd server/api
cp config-example.js config.js
```

Edit `server/api/config.js`:
```javascript
module.exports = {
  // Database Configuration
  database: {
    user: 'firewallcafe',
    host: 'localhost',
    database: 'firewallcafe',
    password: 'your_password',
    port: 5432,
  },
  
  // API Keys
  serperApiKey: 'your_serper_api_key',
  googleTranslateApiKey: 'your_translate_key',
  
  // Server Configuration
  port: 11458,
  corsOrigins: ['http://localhost:3000', 'http://localhost:8080'],
  
  // Security
  sharedSecret: 'your_secret_key_here',
  
  // Features
  enableCache: true,
  cacheTimeout: 300000, // 5 minutes
};
```

## Database Setup

### 1. Create Database and User

```bash
# Access PostgreSQL
psql -U postgres

# Create database and user
CREATE DATABASE firewallcafe;
CREATE USER firewallcafe WITH ENCRYPTED PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE firewallcafe TO firewallcafe;

# Grant schema permissions
\c firewallcafe
GRANT ALL ON SCHEMA public TO firewallcafe;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO firewallcafe;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO firewallcafe;

# Exit psql
\q
```

### 2. Initialize Database Schema

```bash
cd server/api/db_resources

# Run the schema setup
psql -U firewallcafe -d firewallcafe -f config.sql

# Optional: Load test data
psql -U firewallcafe -d firewallcafe -f test_inserts.sql
```

### 3. Verify Database Setup

```bash
# Connect to database
psql -U firewallcafe -d firewallcafe

# Check tables
\dt

# You should see:
# - searches
# - images
# - have_votes
# - votes

# Test query
SELECT COUNT(*) FROM searches;

\q
```

## API Configuration

### 1. Serper.dev Setup (Google Search API)

1. Sign up at [serper.dev](https://serper.dev)
2. Get your API key from the dashboard
3. Add the key to your environment files

### 2. Google Translate API

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Enable the Translation API
3. Create credentials (API key)
4. Add restrictions if needed (IP or referrer)
5. Add the key to your environment files

### 3. Optional: Baidu API Setup

For Baidu search integration, you'll need to configure the custom Baidu integration in `server/api/services/baidu.js`.

## Running the Application

### Development Mode (Recommended)

```bash
# From the client directory
cd client
yarn dev

# This starts:
# - React dev server on http://localhost:3000
# - Express proxy on http://localhost:8080
# - Hot module replacement enabled
```

### Running Services Separately

If you need to run services independently:

#### Frontend Only
```bash
cd client
yarn start
```

#### Express Proxy Server
```bash
cd client
node index.js
```

#### Backend API Server
```bash
cd server/api
node server.js
```

## Verification

### 1. Check Frontend
- Open http://localhost:3000
- You should see the Firewall Cafe homepage
- Try a search to test the integration

### 2. Check Express Proxy
```bash
curl http://localhost:8080/api/health
# Should return: {"status":"ok"}
```

### 3. Check Backend API
```bash
curl http://localhost:11458/dashboard
# Should return dashboard statistics
```

### 4. Test Search Functionality
1. Go to http://localhost:3000
2. Enter a search term
3. Verify results appear from both Google and Baidu
4. Check that translation works

### 5. Test Database Connection
```bash
# Run a test query through the API
curl http://localhost:8080/api/searches?limit=1
```

## Common Issues

### Issue: Node version mismatch
**Solution:**
```bash
nvm use 20
# Or check .nvmrc for the correct version
```

### Issue: PostgreSQL connection refused
**Solution:**
```bash
# Check if PostgreSQL is running
brew services list  # macOS
sudo systemctl status postgresql  # Linux

# Start if needed
brew services start postgresql@14  # macOS
sudo systemctl start postgresql  # Linux
```

### Issue: CORS errors in browser
**Solution:**
- Check that corsOrigins in config.js includes your frontend URL
- Ensure the proxy server is running on port 8080

### Issue: API keys not working
**Solution:**
- Verify keys are correctly set in .env files
- Check API quotas and limits
- Ensure keys have necessary permissions

### Issue: Database permission denied
**Solution:**
```sql
-- Re-grant permissions
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO firewallcafe;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO firewallcafe;
```

## Development Tools

### Useful Commands

```bash
# Lint your code
yarn lint
yarn lint:fix

# Format code
yarn format
yarn format:check

# Run tests
yarn test

# Build for production
yarn build

# Database backup
pg_dump firewallcafe > backup.sql

# Database restore
psql firewallcafe < backup.sql
```

### Browser Extensions

- React Developer Tools
- Redux DevTools (if using Redux)
- Chrome DevTools

### VS Code Extensions

Recommended extensions for development:
- ESLint
- Prettier
- ES7+ React/Redux/React-Native snippets
- Tailwind CSS IntelliSense
- PostgreSQL by Chris Kolkman

### Database Management

For GUI database management:
```bash
# Install TablePlus (macOS)
brew install --cask tableplus

# Or pgAdmin
brew install --cask pgadmin4
```

## Next Steps

Once your development environment is set up:

1. Read the [Architecture Overview](./ARCHITECTURE.md)
2. Review [Contributing Guidelines](./development/CONTRIBUTING.md)
3. Explore the [API Documentation](./api/README.md)
4. Check the [Product Roadmap](../.agent-os/product/roadmap.md)

## Getting Help

If you encounter issues:

1. Check the [Common Issues](#common-issues) section
2. Search existing [GitHub Issues](https://github.com/FIREWALL-cafe/firewall-cafe/issues)
3. Ask in the development Slack channel
4. Create a new issue with details about your environment

## Additional Resources

- [React Documentation](https://react.dev)
- [Express.js Guide](https://expressjs.com/en/guide/routing.html)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/14/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

---

Happy coding! 🚀