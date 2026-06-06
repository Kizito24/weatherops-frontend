# WeatherOps Frontend - Deployment Guide

## Overview

This guide covers deploying the WeatherOps frontend to various environments.

## Deployment Pipeline

```
┌──────────────┐
│ Git Push     │
└──────┬───────┘
       │
       ▼
┌──────────────────────────┐
│ GitHub Actions CI/CD     │
│ - Run tests              │
│ - Build                  │
│ - Security scan          │
└──────┬───────────────────┘
       │
       ├─ Tests Fail ──→ Notify Developer
       │
       ├─ Success ──┐
       │            │
       ├─ Manual Approval (Production)
       │            │
       └────────┬───┘
                │
       ┌────────┴────────────────────┐
       │                             │
       ▼                             ▼
   ┌─────────────┐          ┌─────────────┐
   │ Staging     │          │ Production  │
   │ (Vercel)    │          │ (Vercel)    │
   │ Auto-Deploy │          │ Manual-Approval
   └─────────────┘          └─────────────┘
```

## Environments

### Development
- **URL**: http://localhost:3000
- **Backend**: http://localhost:8000
- **Deploy**: Manual (local)

### Staging
- **URL**: https://weatherops-staging.vercel.app
- **Backend**: https://api-staging.weatherops.com
- **Deploy**: Automatic on `staging` branch

### Production
- **URL**: https://weatherops.com
- **Backend**: https://api.weatherops.com
- **Deploy**: Manual approval on `main` branch

---

## Pre-Deployment Checklist

Before deploying, verify:

```bash
# ✅ All tests pass
npm run test:e2e
npm run test:e2e:headed  # Optional: manual verification

# ✅ Type checking passes
npm run lint

# ✅ Build completes without errors
npm run build

# ✅ No console errors
npm run preview

# ✅ Environment variables are set correctly
cat .env
# Verify:
# - VITE_API_BASE_URL is correct for environment
# - All API keys are present
# - Feature flags are appropriate

# ✅ Git repository is clean
git status
# Should show: "nothing to commit, working tree clean"

# ✅ Branch is up to date
git pull origin main
git log --oneline -5  # Recent commits
```

---

## Local Build Testing

### Build the Application

```bash
# Create production build
npm run build

# Output directory: dist/
# Files generated:
# - HTML files
# - CSS bundles
# - JavaScript chunks
# - Assets (images, fonts)

# Check bundle size
ls -lh dist/
du -sh dist/

# Typical sizes:
# - index.html: 5-10KB
# - CSS bundle: 50-100KB
# - JS bundles: 200-400KB
# - Total: 300-500KB (gzipped)
```

### Test Production Build Locally

```bash
# Preview the build locally
npm run preview

# Output:
#   ➜  Local:   http://localhost:4173/
#
# Test at http://localhost:4173/
# Verify:
# - Page loads correctly
# - Styling looks correct
# - Dark mode works
# - All pages load
# - Console has no errors
```

---

## Deployment to Vercel

### Initial Setup (One-Time)

```bash
# Install Vercel CLI
npm install -g vercel

# Login to Vercel
vercel login

# Link project (run in project root)
vercel link

# Configuration will be created in vercel.json
cat vercel.json
```

### vercel.json Configuration

```json
{
  "buildCommand": "npm install && npm run build",
  "outputDirectory": "dist",
  "framework": "vite",
  "env": {
    "VITE_API_BASE_URL": "@api_base_url",
    "VITE_API_TIMEOUT": "@api_timeout",
    "VITE_WEATHER_API_KEY": "@weather_api_key",
    "VITE_GOOGLE_GENAI_API_KEY": "@genai_api_key"
  },
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}
```

### Deploy to Vercel

```bash
# Deploy to staging environment
vercel --prod --environment staging

# Deploy to production (requires approval)
vercel --prod

# Verify deployment
vercel ls
# Shows all deployments and their status
```

### Set Environment Variables in Vercel

```bash
# Via CLI
vercel env add VITE_API_BASE_URL
# Enter value: https://api-staging.weatherops.com

vercel env add VITE_WEATHER_API_KEY
# Enter value: (your API key)

# Or via Vercel Dashboard:
# 1. Go to project settings
# 2. Environment Variables
# 3. Add each variable for different environments:
#    - Development
#    - Preview
#    - Production
```

---

## GitHub Actions CI/CD

### Workflow File

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main, staging, develop]
  pull_request:
    branches: [main, staging, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    name: Test
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run tests
        run: npm run test:e2e
        env:
          VITE_API_BASE_URL: http://localhost:8000
      
      - name: Build
        run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    name: Deploy to Vercel
    if: github.event_name == 'push' && github.ref != 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        uses: vercel/actions/deploy-production@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}

  deploy-prod:
    needs: test
    runs-on: ubuntu-latest
    name: Deploy to Production
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Request approval
        uses: slack-notify@v1
        with:
          message: 'Production deployment ready for approval'
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
      
      - name: Wait for approval
        uses: trstringer/manual-approval@v1
        with:
          secret: ${{ github.TOKEN }}
          approvers: 'team-leads'
      
      - name: Deploy to Vercel Production
        uses: vercel/actions/deploy-production@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
      
      - name: Notify Slack
        uses: slack-notify@v1
        if: always()
        with:
          message: 'Deployment to production completed'
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
```

### GitHub Actions Secrets

Set these secrets in GitHub repository settings:

```
VERCEL_TOKEN              - Vercel API token
VERCEL_ORG_ID            - Vercel organization ID
VERCEL_PROJECT_ID        - Vercel project ID
SLACK_WEBHOOK            - Slack webhook for notifications
```

---

## Deployment Process

### Step 1: Create Release Branch

```bash
# From main branch
git checkout main
git pull origin main

# Create release branch
git checkout -b release/v1.2.0

# Update version in package.json
# npm version patch  # or minor/major
npm version minor  # e.g., 1.1.0 → 1.2.0

# Commit version bump
git add package.json package-lock.json
git commit -m "chore: bump version to 1.2.0"

# Push release branch
git push origin release/v1.2.0
```

### Step 2: Create Pull Request

```bash
# GitHub will run tests automatically
# After tests pass, create PR for code review

gh pr create \
  --title "Release v1.2.0" \
  --body "$(cat <<EOF
## Release Notes
- Feature: Add CSV export
- Fix: Login validation bug
- Improve: Performance optimization

## Testing
- E2E tests: ✅ Passed
- Manual testing: ✅ Verified
- Performance: ✅ Within budget
EOF
)" \
  --base main \
  --head release/v1.2.0
```

### Step 3: Code Review

```
- Team reviews changes
- Tests must pass
- Performance benchmarks approved
- Security scan passes
```

### Step 4: Merge to Main

```bash
# Approve and merge PR
gh pr merge release/v1.2.0 --merge

# Or manually via GitHub interface
# - Click "Merge pull request"
# - Confirm merge
```

### Step 5: Create Git Tag

```bash
# Tag the release
git tag -a v1.2.0 -m "Release version 1.2.0"

# Push tag to trigger deployment
git push origin v1.2.0

# View tags
git tag -l
git describe --tags
```

### Step 6: GitHub Actions Deploy

```
✅ Tests run on merge
✅ Build completes
✅ Staging deployment automatic
⏳ Production deployment awaits approval
✅ Team approves production deployment
✅ Production deployment starts
✅ Deployment completes
✅ Health checks pass
✅ Monitoring alerts enabled
```

---

## Monitoring & Health Checks

### Post-Deployment Verification

```bash
# Check deployment status
vercel ls

# View deployment logs
vercel logs <url>

# Test production endpoints
curl https://weatherops.com/api/health
curl https://weatherops.com/

# Check error tracking
# - Sentry dashboard
# - Error logs
# - Performance metrics
```

### Health Check Endpoint

```typescript
// Backend should provide health check
GET /api/health

Response: 200 OK
{
  "status": "healthy",
  "timestamp": "2026-06-06T10:00:00Z",
  "version": "1.2.0",
  "database": "connected"
}
```

### Key Metrics to Monitor

```
✅ Page Load Time: < 3s
✅ API Response Time: < 500ms
✅ Error Rate: < 0.1%
✅ Uptime: > 99.9%
✅ Bundle Size: < 500KB
✅ Lighthouse Score: > 90
```

---

## Rollback Procedures

### If Deployment Fails

```bash
# View deployments
vercel ls

# Identify last stable deployment
# Example: v1.1.0-prev

# Rollback to previous version
vercel promote <deployment-url>

# Or manually
vercel --prod --env production --token <token> <previous-build>

# Verify rollback
curl https://weatherops.com/
# Check that page loads with previous version
```

### Rollback Process

```
1. Identify Issue
   - Monitor alerts
   - Check error rates
   - Review user reports

2. Initiate Rollback
   - Select previous stable deployment
   - Click "Promote to Production"
   - Confirm action

3. Verify
   - Check application functionality
   - Monitor error rates
   - Confirm with team

4. Post-Mortem
   - Document what went wrong
   - Schedule incident review
   - Plan fixes
```

---

## Environment-Specific Configuration

### Staging Environment

```env
VITE_API_BASE_URL=https://api-staging.weatherops.com
VITE_APP_ENV=staging
VITE_DEBUG=true
VITE_ENABLE_TREE_ANALYSIS=true
VITE_ENABLE_AI_SUMMARY=true
```

### Production Environment

```env
VITE_API_BASE_URL=https://api.weatherops.com
VITE_APP_ENV=production
VITE_DEBUG=false
VITE_ENABLE_TREE_ANALYSIS=true
VITE_ENABLE_AI_SUMMARY=true
```

---

## Domain & SSL Configuration

### Vercel Auto-HTTPS

Vercel automatically:
- Generates SSL certificates
- Handles renewals
- Enforces HTTPS

### Custom Domain Setup

```bash
# Add custom domain in Vercel dashboard
# 1. Project settings → Domains
# 2. Enter domain: weatherops.com
# 3. Update DNS records (Vercel provides)

# DNS Records to add:
# Type: CNAME
# Name: weatherops.com
# Value: cname.vercel-dns.com

# Verify DNS propagation
nslookup weatherops.com
dig weatherops.com

# Test HTTPS
curl -I https://weatherops.com/
# Should see: HTTP/2 200
```

---

## Performance Optimization for Production

### Build Optimization

```bash
# Analyze bundle size
npm run build -- --analyze

# Check generated chunks
ls -lh dist/assets/

# Expected output:
# - Main bundle: < 200KB
# - CSS: < 50KB
# - Additional chunks: < 100KB each
```

### Caching Strategy

```javascript
// vercel.json - cache headers
{
  "headers": [
    {
      "source": "/assets/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    },
    {
      "source": "/index.html",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=0, must-revalidate"
        }
      ]
    }
  ]
}
```

### Image Optimization

```typescript
// Use next-gen image formats
// Vercel automatically optimizes images

// Example
<img
  src="image.webp"
  alt="Description"
  loading="lazy"
  width="400"
  height="300"
/>
```

---

## Disaster Recovery

### Backup Strategy

```
Daily automated backups:
├─ Database snapshots
├─ Configuration backups
├─ Build artifacts
└─ Git repository mirrors

Recovery Time Objective (RTO): 1 hour
Recovery Point Objective (RPO): 1 hour
```

### Database Backup Verification

```bash
# Test restore from backup weekly
# - Restore to separate environment
# - Run smoke tests
# - Verify data integrity
# - Document process
```

---

## Deployment Troubleshooting

### Build Fails

```bash
# Check build logs
vercel logs

# Common issues:
# 1. Missing environment variables
vercel env list

# 2. Node version mismatch
node --version  # Should be 18+

# 3. Dependency issues
npm ci --production
npm run build
```

### Deployment Timeouts

```bash
# Increase timeout in vercel.json
{
  "buildCommand": "npm install && npm run build",
  "buildCommand": "cd . && npm install && npm run build",
  "gitCommitSha": "HEAD",
  "functions": {
    "api/**": {
      "maxDuration": 30
    }
  }
}
```

### Environment Variable Issues

```bash
# Verify variables in Vercel
vercel env list

# Add missing variable
vercel env add VARIABLE_NAME

# Or update in dashboard
# - Project settings
# - Environment Variables
# - Select environment
# - Edit or add variable
```

---

## Checklist Before Production Deployment

- [ ] All tests passing
- [ ] Code reviewed and approved
- [ ] Performance benchmarks met
- [ ] Security scan passing
- [ ] Environment variables verified
- [ ] Database migrations applied
- [ ] Backup created
- [ ] Rollback plan documented
- [ ] Monitoring configured
- [ ] Team notified
- [ ] Incident response ready
- [ ] Deployment scheduled for low-traffic window

---

**Last Updated**: June 2026  
**Version**: 1.0.0

**Related Documents**:
- [SETUP.md](./SETUP.md) - Local development setup
- [DEVELOPMENT.md](./DEVELOPMENT.md) - Development best practices
- [TESTING.md](./TESTING.md) - Testing strategies
