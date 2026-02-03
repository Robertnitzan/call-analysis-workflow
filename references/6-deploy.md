# Step 6: Deploy to Vercel

## Prerequisites

- Vercel CLI installed: `npm i -g vercel`
- Logged in: `vercel login`

## Build

```bash
cd dashboard
npm run build
```

Output goes to `dist/` folder.

## Copy Data

The analysis results need to be in the build output:

```bash
cp -r public/data dist/
```

This ensures `results.json` is accessible at `/data/results.json`.

## Deploy

```bash
cd dist
vercel --prod --yes
```

### Flags
- `--prod` - Deploy to production (not preview)
- `--yes` - Auto-confirm prompts

## First Deployment

On first deploy, Vercel will ask for project settings:
- Set up and deploy: `Y`
- Scope: Select your account
- Link to existing project: `N`
- Project name: `customer-call-analyzer` (or custom)
- Directory: `./` (current)
- Override settings: `N`

## Subsequent Deployments

After first deploy, just run:

```bash
vercel --prod --yes
```

It will use the saved `.vercel/` configuration.

## Custom Domain (Optional)

```bash
vercel domains add your-customer.call-analyzer.com
```

Or configure in Vercel dashboard.

## Project Configuration

Create `vercel.json` for consistent settings:

```json
{
  "name": "customer-call-analyzer",
  "public": true
}
```

## Output URLs

After deployment:
- Production: `https://project-name.vercel.app`
- Inspect: Link to deployment details
- Alias: Custom domain if configured

## Password Protection (Optional)

For private dashboards, use Vercel's password protection:

1. Go to Project Settings → Deployment Protection
2. Enable "Password Protection"
3. Set password

Or use edge middleware for custom auth.

## Environment Variables

If dashboard needs env vars (unlikely for static):

```bash
vercel env add VARIABLE_NAME
```

## CI/CD Integration

For automated deployments:

```yaml
# .github/workflows/deploy.yml
name: Deploy Dashboard
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run build
      - run: cp -r public/data dist/
      - run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

## Full Deploy Script

```bash
#!/bin/bash
# deploy.sh

set -e

echo "Building dashboard..."
npm run build

echo "Copying data..."
cp -r public/data dist/

echo "Deploying to Vercel..."
cd dist
vercel --prod --yes

echo "Done!"
```

Make executable and run:
```bash
chmod +x deploy.sh
./deploy.sh
```
