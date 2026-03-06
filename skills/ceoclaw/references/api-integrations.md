# API Integrations Reference

This document provides detailed integration guides for all external services used by CEOClaw.

## Vercel Deployment API

### Authentication

```bash
export CEOCLAW_VERCEL_TOKEN="your-token-here"
```

Get token: https://vercel.com/account/tokens

### CLI Alternative

Install Vercel CLI as fallback:
```bash
npm install -g vercel
vercel login
```

The tool will automatically use CLI if API fails.

### API Endpoint

```
POST https://api.vercel.com/v13/deployments
Authorization: Bearer <token>
```

### Payload

```json
{
  "name": "project-name",
  "files": [
    {
      "file": "index.html",
      "data": "<base64-encoded-content>"
    }
  ],
  "target": "production"
}
```

### Response

```json
{
  "id": "deployment-id",
  "url": "project-xyz.vercel.app",
  "readyState": "READY"
}
```

## SMTP Email

### Gmail Configuration

1. **Enable 2-Factor Authentication**
   - Go to Google Account → Security
   - Turn on 2FA

2. **Generate App Password**
   - Google Account → Security → App Passwords
   - Select "Mail" and "Other (Custom name)"
   - Copy the 16-character password

3. **Set Environment Variables**
   ```bash
   export CEOCLAW_SMTP_HOST="smtp.gmail.com"
   export CEOCLAW_SMTP_USER="your-email@gmail.com"
   export CEOCLAW_SMTP_PASS="your-app-password"
   ```

### SendGrid Alternative

1. **Create Account**: https://sendgrid.com
2. **Generate API Key**: Settings → API Keys → Create API Key
3. **Set Variable**:
   ```bash
   export CEOCLAW_SENDGRID_KEY="SG.xxx"
   ```

### Email Rate Limits

- **Gmail**: 500 emails/day for free accounts, 2000/day for Google Workspace
- **SendGrid**: 100 emails/day on free tier

## OpenAI API

### Models Used

- **GPT-4**: Idea generation, content creation
- **GPT-3.5-turbo**: Fallback for cost savings

### Configuration

```bash
export OPENAI_API_KEY="sk-..."
```

Get key: https://platform.openai.com/api-keys

### Rate Limits

Check your OpenAI dashboard for current limits.

### Cost Optimization

- Use GPT-3.5-turbo for non-critical tasks
- Set max_tokens limits
- Cache responses when appropriate

## Reddit API

### Public Access (No Auth)

CEOClaw uses Reddit's public JSON endpoints:

```
https://www.reddit.com/search.json?q=<query>&limit=25
```

No API key required for read-only access.

### Optional: Reddit App (Higher Limits)

1. Create app: https://www.reddit.com/prefs/apps
2. Get credentials:
   ```bash
   export CEOCLAW_REDDIT_CLIENT_ID="..."
   export CEOCLAW_REDDIT_SECRET="..."
   ```

## HackerNews API

Uses Algolia's public search API:

```
https://hn.algolia.com/api/v1/search?query=<query>
```

No authentication required.

## ProductHunt

CEOClaw uses web scraping (no official API access needed for basic research).

**Note**: ProductHunt's official API requires approval. The tool uses public web pages.

## Google Analytics

### Setup GA4

1. Create property: https://analytics.google.com
2. Get Measurement ID (format: G-XXXXXXXXXX)
3. Add to landing page:
   ```html
   <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
   ```

4. Set environment variable:
   ```bash
   export CEOCLAW_ANALYTICS_ID="G-XXXXXXXXXX"
   ```

### Data API (Optional)

For programmatic access to analytics data, set up:
1. Google Cloud Console
2. Enable Analytics Data API
3. Create service account
4. Download credentials JSON

**Note**: Current implementation uses simple analytics. Full GA integration requires OAuth setup.

## Plausible Analytics

### Lightweight Alternative

1. **Add Site**: https://plausible.io
2. **Install Script** (in landing page):
   ```html
   <script defer data-domain="yourdomain.com" src="https://plausible.io/js/script.js"></script>
   ```

3. **Generate API Key**: Settings → API Tokens
4. **Set Variable**:
   ```bash
   export CEOCLAW_PLAUSIBLE_KEY="..."
   ```

### API Endpoint

```
GET https://plausible.io/api/v1/stats/aggregate
  ?site_id=<domain>
  &period=30d
  &metrics=visitors,pageviews
Authorization: Bearer <token>
```

## Rate Limiting Strategy

All tools implement:
- Delays between requests (1-2 seconds)
- Exponential backoff on errors
- Respect platform rate limits
- User-Agent headers

## Error Handling

Common API errors:

### 401 Unauthorized
- Check API key is set correctly
- Verify key hasn't expired
- Ensure correct environment variable name

### 429 Too Many Requests
- Tool will wait and retry
- Check your rate limit quotas
- Consider upgrading paid tier

### 500 Server Error
- Tool will log and continue
- Temporary platform issues
- Try again later

## Testing Integrations

Test each integration individually:

```bash
# Vercel
node --loader tsx tools/deploy.ts test.html

# Email
node --loader tsx tools/email-outreach.ts recipients.json

# Market Research
node --loader tsx tools/market-research.ts "test topic"
```

## Security Best Practices

1. **Never commit API keys** to version control
2. **Use environment variables** for all secrets
3. **Rotate keys regularly**
4. **Use separate keys** for dev/prod
5. **Monitor API usage** for unexpected activity
6. **Set spending limits** where available

## Cost Monitoring

Approximate costs per run:

- **OpenAI**: $0.50 - $2.00 (depending on iterations)
- **Vercel**: Free tier sufficient for testing
- **SendGrid**: Free tier covers basic outreach
- **Analytics**: Free tiers available

Total estimated: **$1-5 per run** (mostly OpenAI)

## Alternative Services

If a service is down or unavailable:

- **Vercel** → Netlify, Cloudflare Pages
- **SendGrid** → Mailgun, AWS SES
- **Gmail** → Any SMTP provider
- **OpenAI** → Anthropic Claude, Google Gemini (requires code changes)

## Support

For API-specific issues:
- Check provider's status page
- Review provider documentation
- Contact provider support

For CEOClaw integration issues:
- GitHub Issues: https://github.com/openclaw/openclaw/issues
- Discord: https://discord.gg/clawd
