# Deployment Guide

Deploy landing pages to Vercel using the API or CLI for instant public access with custom domains and SSL.

## Overview

Vercel deployment options:
- **API** - Programmatic deployment via REST API
- **CLI** - Command-line deployment with `vercel` command
- **Git** - Auto-deploy from GitHub/GitLab (manual setup)

This guide focuses on API and CLI methods for autonomous deployment.

## Prerequisites

### Vercel Account Setup

1. Create account: https://vercel.com/signup
2. Get API token: https://vercel.com/account/tokens
3. Set environment variable:
```bash
export CEOCLAW_VERCEL_TOKEN="your_token_here"
```

### Install Vercel CLI (Optional)

```bash
npm install -g vercel

# Login
vercel login

# Verify
vercel whoami
```

## Method 1: Vercel API (Recommended)

### Simple File Deployment

```bash
#!/bin/bash
# deploy-vercel-api.sh

set -e

FILE="${1:?Usage: $0 <html-file>}"
TOKEN="${CEOCLAW_VERCEL_TOKEN:?CEOCLAW_VERCEL_TOKEN not set}"

if [ ! -f "$FILE" ]; then
  echo "Error: File not found: $FILE"
  exit 1
fi

# Read file content
CONTENT=$(cat "$FILE" | jq -Rs .)

# Create deployment payload
PAYLOAD=$(jq -n \
  --arg name "$(basename "$FILE" .html)" \
  --arg file "$FILE" \
  --argjson content "$CONTENT" \
  '{
    name: $name,
    files: [
      {
        file: "index.html",
        data: $content
      }
    ],
    projectSettings: {
      framework: null
    }
  }')

# Deploy
RESPONSE=$(curl -sf -X POST "https://api.vercel.com/v13/deployments" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD")

# Extract URL
URL=$(echo "$RESPONSE" | jq -r '.url')

if [ "$URL" = "null" ]; then
  echo "Deployment failed:"
  echo "$RESPONSE" | jq .
  exit 1
fi

echo "✅ Deployed to: https://$URL"
echo "$RESPONSE" | jq '{id, url, readyState}' > deployment-info.json
```

### Multi-File Deployment

For Next.js or sites with multiple files:

```bash
#!/bin/bash
# deploy-nextjs-api.sh

set -e

PROJECT_DIR="${1:?Usage: $0 <project-dir>}"
TOKEN="${CEOCLAW_VERCEL_TOKEN:?CEOCLAW_VERCEL_TOKEN not set}"

cd "$PROJECT_DIR"

# Collect all files
FILES=$(find . -type f ! -path "*/node_modules/*" ! -path "*/.git/*" | while read -r file; do
  RELATIVE=${file#./}
  CONTENT=$(cat "$file" | base64)
  
  jq -n \
    --arg file "$RELATIVE" \
    --arg data "$CONTENT" \
    '{file: $file, data: $data, encoding: "base64"}'
done | jq -s .)

# Create payload
PAYLOAD=$(jq -n \
  --arg name "$(basename "$PROJECT_DIR")" \
  --argjson files "$FILES" \
  '{
    name: $name,
    files: $files,
    projectSettings: {
      framework: "nextjs"
    }
  }')

# Deploy
RESPONSE=$(curl -sf -X POST "https://api.vercel.com/v13/deployments" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD")

URL=$(echo "$RESPONSE" | jq -r '.url')
echo "✅ Deployed to: https://$URL"
```

## Method 2: Vercel CLI

### Simple Deployment

```bash
cd ~/path/to/landing-page

# Deploy to production
vercel --prod

# Or preview deployment (staging)
vercel

# With project name
vercel --name my-startup
```

### Automated CLI Deployment

```bash
#!/bin/bash
# deploy-cli.sh

set -e

FILE="${1:?Usage: $0 <html-file>}"

# Create temp directory
DEPLOY_DIR=$(mktemp -d)
cp "$FILE" "$DEPLOY_DIR/index.html"

cd "$DEPLOY_DIR"

# Deploy
echo "Deploying $FILE..."
OUTPUT=$(vercel --prod --yes 2>&1)

# Extract URL
URL=$(echo "$OUTPUT" | grep -oE 'https://[^ ]+' | head -1)

if [ -z "$URL" ]; then
  echo "Deployment failed"
  echo "$OUTPUT"
  exit 1
fi

echo "✅ Deployed to: $URL"

# Store deployment info
cd ~/.openclaw/ceoclaw
sqlite3 ceoclaw.db <<SQL
INSERT INTO deployments (url, platform, status, deployed_at)
VALUES ('$URL', 'vercel', 'live', datetime('now'));
SQL

# Cleanup
rm -rf "$DEPLOY_DIR"
```

## Custom Domain Setup

### Add Domain via API

```bash
#!/bin/bash
# add-domain.sh

set -e

PROJECT="${1:?Usage: $0 <project-name> <domain>}"
DOMAIN="${2:?}"
TOKEN="${CEOCLAW_VERCEL_TOKEN:?}"

# Add domain to project
curl -sf -X POST "https://api.vercel.com/v10/projects/$PROJECT/domains" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"name\": \"$DOMAIN\"}" | jq .

echo ""
echo "✅ Domain $DOMAIN added to $PROJECT"
echo ""
echo "Next steps:"
echo "1. Add DNS records from response above"
echo "2. Wait for DNS propagation (can take up to 48h)"
echo "3. SSL cert will be provisioned automatically"
```

### DNS Configuration

Point your domain to Vercel:

```
# A Record
Type: A
Name: @
Value: 76.76.21.21

# CNAME Record (www)
Type: CNAME
Name: www
Value: cname.vercel-dns.com
```

Or check project-specific DNS:
```bash
curl -sf "https://api.vercel.com/v10/projects/$PROJECT/domains/$DOMAIN/config" \
  -H "Authorization: Bearer $TOKEN" | jq .
```

## Monitoring Deployment Status

### Check Deployment State

```bash
#!/bin/bash
# check-deployment.sh

set -e

DEPLOYMENT_ID="${1:?Usage: $0 <deployment-id>}"
TOKEN="${CEOCLAW_VERCEL_TOKEN:?}"

# Poll until ready
while true; do
  RESPONSE=$(curl -sf "https://api.vercel.com/v13/deployments/$DEPLOYMENT_ID" \
    -H "Authorization: Bearer $TOKEN")
  
  STATE=$(echo "$RESPONSE" | jq -r '.readyState')
  
  echo "Status: $STATE"
  
  if [ "$STATE" = "READY" ]; then
    echo "✅ Deployment ready!"
    echo "$RESPONSE" | jq '{url, alias, readyState}'
    break
  elif [ "$STATE" = "ERROR" ]; then
    echo "❌ Deployment failed:"
    echo "$RESPONSE" | jq .
    exit 1
  fi
  
  sleep 5
done
```

### List All Deployments

```bash
curl -sf "https://api.vercel.com/v6/deployments" \
  -H "Authorization: Bearer $CEOCLAW_VERCEL_TOKEN" | \
  jq -r '.deployments[] | "\(.name): https://\(.url) (\(.state))"'
```

## Environment Variables

Set environment variables for deployed projects:

```bash
#!/bin/bash
# set-env.sh

set -e

PROJECT="${1:?Usage: $0 <project-id> <key> <value>}"
KEY="${2:?}"
VALUE="${3:?}"
TOKEN="${CEOCLAW_VERCEL_TOKEN:?}"

curl -sf -X POST "https://api.vercel.com/v10/projects/$PROJECT/env" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"key\": \"$KEY\",
    \"value\": \"$VALUE\",
    \"type\": \"encrypted\",
    \"target\": [\"production\"]
  }" | jq .

echo "✅ Environment variable $KEY set"
```

## Rollback to Previous Deployment

```bash
#!/bin/bash
# rollback.sh

PROJECT="${1:?Usage: $0 <project-name> <deployment-id>}"
DEPLOYMENT_ID="${2:?}"
TOKEN="${CEOCLAW_VERCEL_TOKEN:?}"

# Promote deployment to production
curl -sf -X PATCH "https://api.vercel.com/v13/deployments/$DEPLOYMENT_ID/promote" \
  -H "Authorization: Bearer $TOKEN" | jq .

echo "✅ Rolled back to deployment: $DEPLOYMENT_ID"
```

## Complete Deployment Workflow

```bash
#!/bin/bash
# complete-deployment.sh

set -e

IDEA_ID="${1:?Usage: $0 <idea-id>}"
DB=~/.openclaw/ceoclaw/ceoclaw.db
TOKEN="${CEOCLAW_VERCEL_TOKEN:?}"

echo "🚀 Deploying idea #$IDEA_ID..."

# 1. Get idea details
IDEA=$(sqlite3 "$DB" <<SQL
SELECT json_object(
  'title', title,
  'filename', 'landing-' || lower(replace(title, ' ', '-')) || '.html'
)
FROM ideas
WHERE id = $IDEA_ID;
SQL
)

TITLE=$(echo "$IDEA" | jq -r '.title')
FILE=$(echo "$IDEA" | jq -r '.filename')

if [ ! -f "$FILE" ]; then
  echo "Error: Landing page not found: $FILE"
  exit 1
fi

# 2. Deploy to Vercel
echo "📤 Deploying to Vercel..."
CONTENT=$(cat "$FILE" | jq -Rs .)

PAYLOAD=$(jq -n \
  --arg name "$TITLE" \
  --argjson content "$CONTENT" \
  '{
    name: $name,
    files: [{file: "index.html", data: $content}]
  }')

RESPONSE=$(curl -sf -X POST "https://api.vercel.com/v13/deployments" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD")

URL=$(echo "$RESPONSE" | jq -r '.url')
DEPLOYMENT_ID=$(echo "$RESPONSE" | jq -r '.id')

if [ "$URL" = "null" ]; then
  echo "❌ Deployment failed:"
  echo "$RESPONSE" | jq .
  exit 1
fi

# 3. Wait for ready state
echo "⏳ Waiting for deployment..."
sleep 5

while true; do
  STATE=$(curl -sf "https://api.vercel.com/v13/deployments/$DEPLOYMENT_ID" \
    -H "Authorization: Bearer $TOKEN" | jq -r '.readyState')
  
  if [ "$STATE" = "READY" ]; then
    break
  elif [ "$STATE" = "ERROR" ]; then
    echo "❌ Deployment failed"
    exit 1
  fi
  
  sleep 3
done

# 4. Store in database
sqlite3 "$DB" <<SQL
INSERT INTO deployments (idea_id, url, platform, status, deployed_at)
VALUES ($IDEA_ID, 'https://$URL', 'vercel', 'live', datetime('now'));
SQL

DEPLOYMENT_DB_ID=$(sqlite3 "$DB" "SELECT last_insert_rowid();")

echo ""
echo "✅ Deployment successful!"
echo "   URL: https://$URL"
echo "   ID: $DEPLOYMENT_ID"
echo "   DB ID: $DEPLOYMENT_DB_ID"
echo ""
echo "Next steps:"
echo "1. Test the live site: open https://$URL"
echo "2. Set up analytics (see metrics.md)"
echo "3. Start marketing (see marketing.md)"
```

## Alternative Platforms

### Netlify

```bash
# Install CLI
npm install -g netlify-cli

# Deploy
netlify deploy --prod --dir=./dist

# With API
curl -X POST "https://api.netlify.com/api/v1/sites" \
  -H "Authorization: Bearer $NETLIFY_TOKEN" \
  -d @deployment.tar.gz
```

### GitHub Pages

```bash
# Assumes git repo initialized
git add index.html
git commit -m "Deploy landing page"
git push origin main

# Enable Pages in repo settings
# Or via API:
curl -X POST "https://api.github.com/repos/USER/REPO/pages" \
  -H "Authorization: token $GITHUB_TOKEN" \
  -d '{"source": {"branch": "main", "path": "/"}}'
```

### Cloudflare Pages

```bash
# Install Wrangler
npm install -g wrangler

# Deploy
wrangler pages publish ./dist --project-name=my-startup
```

## Best Practices

1. **Always test locally first:**
```bash
python3 -m http.server 8000
# or
npx serve .
```

2. **Use preview deployments for testing:**
```bash
vercel  # No --prod flag
```

3. **Set up redirects if needed:**
Create `vercel.json`:
```json
{
  "redirects": [
    { "source": "/old-page", "destination": "/new-page" }
  ]
}
```

4. **Enable analytics:**
```json
{
  "analytics": {
    "enable": true
  }
}
```

5. **Configure headers:**
```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        }
      ]
    }
  ]
}
```

## Troubleshooting

**401 Unauthorized:**
- Verify token: `curl -H "Authorization: Bearer $TOKEN" https://api.vercel.com/v2/user`
- Token expired? Generate new one
- Scope issues? Check token permissions

**429 Rate Limited:**
- Hobby plan: 100 deployments/day
- Wait or upgrade plan
- Cache deployment responses

**Build fails:**
- Check file paths (case-sensitive)
- Verify all dependencies included
- Review build logs in dashboard

**Domain not working:**
- DNS can take 24-48h to propagate
- Check with: `dig yourdomain.com`
- Ensure nameservers point to registrar

**SSL certificate issues:**
- Vercel auto-provisions certs
- Can take a few minutes after DNS setup
- Check cert status in dashboard

## Cleanup

Remove old deployments to save bandwidth:

```bash
# List all deployments
vercel ls

# Remove specific deployment
vercel rm <deployment-url>

# Or via API
curl -X DELETE "https://api.vercel.com/v13/deployments/$DEPLOYMENT_ID" \
  -H "Authorization: Bearer $TOKEN"
```

## Next Steps

After successful deployment:
1. Verify site loads correctly
2. Test on mobile devices
3. Set up analytics tracking (see `metrics.md`)
4. Launch marketing campaigns (see `marketing.md`)
5. Start email outreach (see `email-outreach.md`)
