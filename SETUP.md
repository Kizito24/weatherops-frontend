# WeatherOps Frontend - Development Setup Guide

## Prerequisites

Before starting, ensure you have the following installed:

### Required
- **Node.js** (v18.0.0 or higher)
- **npm** (v9.0.0 or higher) - comes with Node.js
- **Git** (v2.30.0 or higher)

### Optional
- **VS Code** - Recommended editor
- **Git GUI** - For easier version control
- **Postman** or **Insomnia** - For API testing

## Installation Steps

### Step 1: Clone the Repository

```bash
# Clone the repository
git clone https://github.com/Kizito24/weatherops-frontend.git

# Navigate to project directory
cd weatherops-frontend
```

### Step 2: Install Dependencies

```bash
# Install all npm packages
npm install

# Verify installation
npm list
```

**Expected output**: See package list without errors

### Step 3: Environment Configuration

Create a `.env` file in the root directory:

```bash
# Copy the template (if exists)
cp .env.example .env

# Or create manually
touch .env
```

**Add the following environment variables**:

```env
# Backend API Configuration
VITE_API_BASE_URL=http://localhost:8000
VITE_API_TIMEOUT=30000

# Weather API Configuration
VITE_WEATHER_API_KEY=your_weather_api_key_here
VITE_WEATHER_API_BASE=https://api.openweathermap.org

# Google Generative AI Configuration
VITE_GOOGLE_GENAI_API_KEY=your_google_genai_key_here

# Application Configuration
VITE_APP_NAME=WeatherOps
VITE_APP_ENV=development
VITE_DEBUG=true

# Feature Flags
VITE_ENABLE_TREE_ANALYSIS=true
VITE_ENABLE_AI_SUMMARY=true

# Optional: Database Connection (if running full stack locally)
DATABASE_URL=postgresql://user:password@localhost:5432/weatherops
```

**To get API keys**:
1. **Weather API**: Visit [OpenWeatherMap](https://openweathermap.org/api) and sign up
2. **Google Generative AI**: Visit [Google AI Studio](https://makersuite.google.com/app/apikey)
3. **Backend API URL**: Usually `http://localhost:8000` for local development

### Step 4: Verify Installation

```bash
# Check Node version
node --version
# Output: v18.x.x or higher

# Check npm version
npm --version
# Output: v9.x.x or higher

# Check installed packages
npm list react react-dom vite

# Run type checking
npm run lint

# Output should have no errors
```

## Running the Development Server

### Start Development Server

```bash
# Run the Vite development server
npm run dev

# Output example:
# > vite --port=3000 --host=0.0.0.0
# 
#   VITE v6.2.3  ready in 234 ms
#
#   ➜  Local:   http://localhost:3000/
#   ➜  press h to show help
```

### Access the Application

1. Open your browser
2. Navigate to `http://localhost:3000`
3. You should see the WeatherOps login page

### First-Time Login

```
Email: test@example.com
Password: Test123!

Note: Use credentials from your backend's test data
```

## Project Structure

```
weatherops-frontend/
├── src/
│   ├── main.tsx              # Application entry point
│   ├── App.tsx               # Root component
│   ├── types.ts              # TypeScript type definitions
│   │
│   ├── components/           # Reusable components
│   │   ├── AuthPage.tsx
│   │   ├── OverviewPage.tsx
│   │   ├── WeatherPage.tsx
│   │   ├── LocationsPage.tsx
│   │   ├── RulesPage.tsx
│   │   ├── AlertsPage.tsx
│   │   ├── TreesPage.tsx
│   │   ├── SettingsPage.tsx
│   │   ├── Sidebar.tsx
│   │   ├── Navbar.tsx
│   │   ├── ExportModal.tsx
│   │   ├── FormInput.tsx
│   │   ├── FormSelect.tsx
│   │   ├── KPICard.tsx
│   │   ├── Badge.tsx
│   │   ├── EmptyState.tsx
│   │   ├── Pagination.tsx
│   │   ├── Skeletons.tsx
│   │   └── ErrorBoundary.tsx
│   │
│   ├── lib/                  # Utility functions and API clients
│   │   ├── api/
│   │   │   ├── auth.ts
│   │   │   ├── locations.ts
│   │   │   ├── rules.ts
│   │   │   ├── alerts.ts
│   │   │   ├── weather.ts
│   │   │   └── tree.ts
│   │   ├── auth/
│   │   │   └── tokenManager.ts
│   │   ├── export.ts
│   │   └── validation.ts
│   │
│   └── assets/               # Images and static files
│
├── e2e/                      # End-to-end tests
│   └── *.spec.ts
│
├── public/                   # Public static files
├── index.html                # HTML template
├── vite.config.ts            # Vite configuration
├── tsconfig.json             # TypeScript configuration
├── tailwind.config.js        # Tailwind CSS configuration
├── package.json              # Dependencies and scripts
├── package-lock.json         # Locked dependencies
├── .env                      # Environment variables (local)
├── .env.example              # Environment template
├── .gitignore                # Git ignore rules
└── vercel.json               # Vercel deployment config
```

## Available npm Scripts

```bash
# Development
npm run dev                   # Start development server on port 3000

# Build
npm run build                 # Build for production

# Preview
npm run preview               # Preview production build locally

# Code Quality
npm run lint                  # TypeScript type checking

# Testing
npm run test:e2e             # Run Playwright E2E tests
npm run test:e2e:ui         # Run tests with UI
npm run test:e2e:debug      # Run tests in debug mode
npm run test:e2e:headed     # Run tests in headed mode (see browser)

# Cleanup
npm run clean                # Remove build artifacts
```

## Troubleshooting

### Port Already in Use

```bash
# Error: Port 3000 is already in use
# Solution 1: Stop the process using port 3000
lsof -i :3000
kill -9 <PID>

# Solution 2: Use a different port
npm run dev -- --port 3001
```

### Dependencies Installation Issues

```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Environment Variables Not Loading

```bash
# Check if .env file exists
ls -la .env

# Verify variables are set
echo $VITE_API_BASE_URL

# Restart dev server after changing .env
npm run dev
```

### TypeScript Errors

```bash
# Run type checking
npm run lint

# Fix auto-fixable issues
npm run lint -- --noEmit

# Check specific file
npx tsc --noEmit src/components/AuthPage.tsx
```

### API Connection Issues

```bash
# Verify backend is running
curl http://localhost:8000/health

# Check API base URL in .env
cat .env | grep VITE_API_BASE_URL

# Test API connectivity
curl -X GET http://localhost:8000/api/health \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Node Version Issues

```bash
# Check current Node version
node --version

# Using nvm (Node Version Manager)
nvm install 18
nvm use 18

# Using fnm (Fast Node Manager)
fnm install 18
fnm use 18
```

## Development Workflow

### 1. Create a New Feature Branch

```bash
# Pull latest changes
git pull origin main

# Create feature branch
git checkout -b feature/your-feature-name

# Example
git checkout -b feature/add-export-csv
```

### 2. Make Changes

```bash
# Edit files in your editor
# Changes are automatically reflected in dev server

# Check file changes
git status

# Preview changes
git diff src/components/ExportModal.tsx
```

### 3. Test Your Changes

```bash
# Run type checking
npm run lint

# Run E2E tests
npm run test:e2e

# Manual browser testing at http://localhost:3000
```

### 4. Commit Changes

```bash
# Stage specific files
git add src/components/ExportModal.tsx src/lib/export.ts

# Or stage all changes
git add .

# Commit with descriptive message
git commit -m "feat: add CSV export functionality"

# View commit history
git log --oneline
```

### 5. Push and Create Pull Request

```bash
# Push to remote
git push origin feature/add-export-csv

# Create PR on GitHub (or use git CLI)
gh pr create --title "Add CSV export" \
  --body "Enables exporting alerts and rules as CSV files"
```

## Browser DevTools

### React DevTools
Install the React DevTools browser extension:
- **Chrome**: [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/)
- **Firefox**: [React Developer Tools](https://addons.mozilla.org/firefox/addon/react-devtools/)

### Usage
1. Open DevTools (F12)
2. Go to "Components" tab
3. Inspect components and their props/state in real-time

### Redux DevTools (if using Redux)
```bash
npm install --save-dev redux-devtools
```

## Hot Module Replacement (HMR)

The dev server supports HMR - changes to files are reflected instantly without full page refresh.

```typescript
// Example: Editing this component
// src/components/KPICard.tsx

export default function KPICard({value}) {
  return (
    <div>
      <h2>{value}</h2>  // Change this text
    </div>
  );
}

// Save file → Changes appear instantly in browser
```

## Database Setup (Optional - Full Stack Development)

If developing with a local backend:

```bash
# Create PostgreSQL database
createdb weatherops_dev

# Run migrations (if using backend)
cd ../weatherops-backend
npm run migrate

# Seed test data
npm run seed
```

## Performance Tips

### 1. Enable Source Maps (Already Enabled in Dev)
```typescript
// vite.config.ts - Already configured
export default {
  define: {
    'process.env.DEBUG': true,
  }
}
```

### 2. Use React DevTools Profiler
- Open React DevTools
- Go to "Profiler" tab
- Record component renders
- Identify performance bottlenecks

### 3. Check Network Performance
- Open DevTools → Network tab
- Look for slow API calls
- Monitor bundle sizes

### 4. Lighthouse Audit
```bash
# Built-in Chrome Lighthouse
# DevTools → Lighthouse → Generate report
```

## VS Code Configuration

### Recommended Extensions
```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "firsttimersonly.first-timers-only",
    "firefox-devtools.vscode-firefox-debug",
    "bradlc.vscode-tailwindcss",
    "orta.vscode-twoslash-queries",
    "mtxr.sqltools",
    "typescript.tsc-problem-matcher"
  ]
}
```

### Workspace Settings
Create `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "files.exclude": {
    "**/node_modules": true,
    "**/.next": true,
    "**/dist": true
  }
}
```

## Security Best Practices

### 1. Sensitive Data in .env
```env
# ✅ DO: Store sensitive keys in .env
VITE_API_KEY=secret_key_here

# ❌ DON'T: Commit .env file
# Already in .gitignore

# ✅ DO: Use .env.example for template
VITE_API_KEY=your_api_key_here
```

### 2. API Token Security
```typescript
// ✅ Secure: Token stored in httpOnly cookie (backend handles)
// ✅ Secure: Token in localStorage with reasonable expiration

// ❌ Insecure: Exposing tokens in URLs or logs
// ❌ Insecure: Long token expiration times
```

### 3. CORS Configuration
```typescript
// Backend should configure CORS for localhost
// Frontend should only make requests to configured API URLs
const API_BASE_URL = process.env.VITE_API_BASE_URL;
```

## Getting Help

### Documentation
- [DOCUMENTATION_INDEX.md](./DOCUMENTATION_INDEX.md) - Navigation guide
- [ARCHITECTURE.md](./ARCHITECTURE.md) - System design
- [DEVELOPMENT.md](./DEVELOPMENT.md) - Development practices
- [COMPONENTS.md](./COMPONENTS.md) - Component guide

### Resources
- [Vite Docs](https://vitejs.dev/)
- [React Docs](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Tailwind CSS Docs](https://tailwindcss.com/docs)

### Slack/Discord
- Ask in #development channel
- Tag @backend-team if API issue
- Tag @devops if infrastructure issue

### Issues
- Create GitHub issue with:
  - Detailed description
  - Steps to reproduce
  - Expected vs actual behavior
  - Environment info (Node version, OS, etc)

---

**Last Updated**: June 2026  
**Version**: 1.0.0

**Next Steps**: Read [DEVELOPMENT.md](./DEVELOPMENT.md) for coding standards and best practices.
