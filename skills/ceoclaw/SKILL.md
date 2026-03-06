---
name: ceoclaw
description: "Autonomous AI founder that launches and operates small internet businesses through market research, product ideation, landing page creation, deployment, marketing campaigns, user outreach, and metrics tracking. Use when: starting a business, testing startup ideas, automating business operations, or running growth experiments. WARNING: Performs real-world actions (deployments, emails, social posts). Always use approval mode."
metadata:
  {
    "openclaw":
      {
        "emoji": "👔",
        "requires": { "anyBins": ["curl", "node"], "env": ["OPENAI_API_KEY"] },
        "homepage": "https://github.com/openclaw/openclaw/"
      }
  }
---

# CEOClaw — Autonomous AI Founder

Launch and operate small internet businesses autonomously through an 8-phase execution workflow. This skill guides you through market research, ideation, product creation, deployment, marketing, outreach, and metrics tracking using real APIs and web scraping.

## ⚠️ Safety & Approval Requirements

This skill performs **real-world actions** with consequences:
- Deploys live websites to the internet (Vercel)
- Sends emails to real people (SMTP)
- Posts marketing content publicly
- Consumes API credits and spending

**ALWAYS:**
- Run in dry-run mode first to preview actions
- Get explicit approval before live deployments
- Review generated content before publishing
- Start with test/staging environments
- Monitor costs and rate limits

See `references/safety-guidelines.md` for complete guidelines.

## When to Use

✅ **Use CEOClaw for:**
- Launching MVP products and testing startup ideas
- Automating market research and validation
- Building simple web products (landing pages, waitlists)
- Running marketing experiments and growth tests
- Tracking early-stage metrics

❌ **Don't use for:**
- Businesses requiring legal/compliance review
- Handling sensitive user data
- Large-scale enterprise operations
- Instant results (building takes time and iteration)

## Prerequisites

### Required Environment Variables

```bash
# Essential
export OPENAI_API_KEY="sk-..."                    # GPT-4 for content generation

# Deployment (at least one)
export CEOCLAW_VERCEL_TOKEN="..."                 # Vercel API token
# OR use Vercel CLI: npm i -g vercel && vercel login

# Email Outreach (at least one)
export CEOCLAW_SMTP_HOST="smtp.gmail.com"         # SMTP server
export CEOCLAW_SMTP_PORT="587"                    # SMTP port (587/465)
export CEOCLAW_SMTP_USER="you@example.com"        # Email account
export CEOCLAW_SMTP_PASS="..."                    # App password
# OR
export CEOCLAW_SENDGRID_KEY="SG...."              # SendGrid API key

# Optional
export CEOCLAW_ANALYTICS_ID="G-..."               # Google Analytics 4
export CEOCLAW_PLAUSIBLE_DOMAIN="yourdomain.com"  # Plausible domain
export CEOCLAW_REDDIT_CLIENT_ID="..."             # Reddit API
export CEOCLAW_REDDIT_SECRET="..."                # Reddit API secret
```

### Required Binaries

- `curl` or `node` (for HTTP requests)
- `node` ≥22 (for running generated code)
- `vercel` CLI (optional, for deployments)
- `sqlite3` (optional, for inspecting database)

## Core Workflow

CEOClaw follows an 8-phase autonomous founder loop:

### Phase 1: Market Discovery
Scrape Reddit, HackerNews, and ProductHunt for user problems and pain points.

### Phase 2: Idea Generation  
Use GPT-4 to synthesize startup ideas from market signals with scoring and validation.

### Phase 3: Validation
Check demand signals through keyword volume, community engagement, and existing solutions.

### Phase 4: Product Build
Generate responsive landing pages with copy, design, and conversion elements.

### Phase 5: Deployment
Deploy to Vercel (or other platforms) with custom domains and SSL.

### Phase 6: Marketing
Create and distribute content across channels (blog, social, email, ads).

### Phase 7: User Outreach
Send personalized emails to early adopters and potential customers.

### Phase 8: Analytics & Iteration
Track metrics (visits, signups, conversions) and iterate based on data.

## Quick Start

### 1. Initialize Business Session

Create a tracking database to store research, ideas, and metrics:

```bash
mkdir -p ~/.openclaw/ceoclaw
cd ~/.openclaw/ceoclaw

# Create SQLite database
sqlite3 ceoclaw.db <<EOF
CREATE TABLE IF NOT EXISTS market_signals (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  source TEXT NOT NULL,
  problem TEXT NOT NULL,
  pain_level INTEGER,
  keywords TEXT,
  url TEXT,
  engagement INTEGER,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS ideas (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  description TEXT,
  target_market TEXT,
  solution TEXT,
  score INTEGER,
  status TEXT DEFAULT 'draft',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS deployments (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  idea_id INTEGER,
  url TEXT NOT NULL,
  platform TEXT,
  status TEXT,
  deployed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (idea_id) REFERENCES ideas(id)
);

CREATE TABLE IF NOT EXISTS metrics (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  deployment_id INTEGER,
  date DATE,
  visits INTEGER DEFAULT 0,
  signups INTEGER DEFAULT 0,
  conversions INTEGER DEFAULT 0,
  revenue REAL DEFAULT 0,
  FOREIGN KEY (deployment_id) REFERENCES deployments(id)
);
EOF

echo "✅ Database initialized at ~/.openclaw/ceoclaw/ceoclaw.db"
```

### 2. Run Market Research

Use references for detailed workflows:
- See `references/market-research.md` for scraping patterns
- See `references/api-integrations.md` for API authentication

### 3. Generate & Validate Ideas

Follow guidance in `references/idea-generation.md` for prompting strategies.

### 4. Build & Deploy Product

See `references/landing-pages.md` and `references/deployment.md` for step-by-step instructions.

### 5. Launch Marketing & Outreach

See `references/marketing.md` and `references/email-outreach.md` for campaign workflows.

### 6. Track Metrics

See `references/metrics.md` for analytics integration patterns.

## References

All detailed workflows live in `{baseDir}/references/`:

- **market-research.md** — Reddit, HN, ProductHunt scraping patterns
- **idea-generation.md** — GPT-4 prompts for startup ideation
- **landing-pages.md** — HTML/React generation with Tailwind CSS
- **deployment.md** — Vercel API deployment + CLI fallback
- **marketing.md** — Content generation for blog/social/email/ads
- **email-outreach.md** — SMTP setup and personalized campaigns
- **metrics.md** — Google Analytics and Plausible integration
- **business-workflow.md** — Complete 8-phase orchestration
- **api-integrations.md** — API authentication guides
- **safety-guidelines.md** — Ethics, approval gates, compliance

## Usage Examples

### Example 1: Full Business Launch

```bash
# Ask the agent:
"Use CEOClaw to launch a business in the developer tools space. 
Start with market research, generate 3 ideas, pick the best one, 
build a landing page, deploy to Vercel, and set up email outreach. 
Run in approval mode - ask before each deployment."
```

### Example 2: Market Research Only

```bash
# Ask the agent:
"Use CEOClaw to research SaaS problems on Reddit and HackerNews. 
Focus on /r/SaaS, /r/startups, and HN 'Ask HN' posts from the last month. 
Store findings in the database and summarize the top 5 pain points."
```

### Example 3: Deploy Existing Idea

```bash
# Ask the agent:
"I have a startup idea: 'Email warm-up service for cold outreach'.
Use CEOClaw to build a landing page, deploy it to Vercel, 
and create a Twitter thread announcing the launch."
```

## Workflow Orchestration

For autonomous multi-phase execution, follow `references/business-workflow.md` which provides:
- Phase sequencing and dependencies
- Error handling and retries
- Approval gates and dry-run modes
- Progress tracking and logging
- Iteration and improvement loops

## Troubleshooting

### API Rate Limits
- Reddit: 60 requests/minute (unauthenticated), 100/minute (authenticated)
- HackerNews: No official limit, be respectful
- OpenAI: Depends on tier (check dashboard)
- Vercel: 100 deployments/day (Hobby), unlimited (Pro)

### Common Issues

**"Database locked"**: Close other SQLite connections
```bash
fuser ~/.openclaw/ceoclaw/ceoclaw.db  # Check processes
```

**"Vercel deployment failed"**: Check token permissions
```bash
curl -H "Authorization: Bearer $CEOCLAW_VERCEL_TOKEN" \
  https://api.vercel.com/v2/user
```

**"SMTP authentication failed"**: Use app-specific password (Gmail)
- Gmail: https://myaccount.google.com/apppasswords
- Must have 2FA enabled first

**"Reddit API 429"**: Add authentication or slow down requests
```bash
# Test Reddit auth
curl -A "CEOClaw/1.0" -u "$CEOCLAW_REDDIT_CLIENT_ID:$CEOCLAW_REDDIT_SECRET" \
  -d "grant_type=client_credentials" \
  https://www.reddit.com/api/v1/access_token
```

## Database Schema

Track business experiments in `~/.openclaw/ceoclaw/ceoclaw.db`:

```sql
-- Market signals from research
market_signals(id, source, problem, pain_level, keywords, url, engagement, created_at)

-- Generated business ideas
ideas(id, title, description, target_market, solution, score, status, created_at)

-- Live deployments
deployments(id, idea_id, url, platform, status, deployed_at)

-- Performance metrics
metrics(id, deployment_id, date, visits, signups, conversions, revenue)
```

Query examples:
```bash
# Top problems by engagement
sqlite3 ~/.openclaw/ceoclaw/ceoclaw.db \
  "SELECT problem, engagement FROM market_signals ORDER BY engagement DESC LIMIT 10"

# Ideas by score
sqlite3 ~/.openclaw/ceoclaw/ceoclaw.db \
  "SELECT title, score FROM ideas ORDER BY score DESC"

# Deployment performance
sqlite3 ~/.openclaw/ceoclaw/ceoclaw.db \
  "SELECT d.url, SUM(m.visits) as total_visits 
   FROM deployments d 
   LEFT JOIN metrics m ON m.deployment_id = d.id 
   GROUP BY d.id"
```

## Best Practices

1. **Always start with dry-run mode** to preview actions
2. **Use test domains** for initial deployments (*.vercel.app)
3. **Review all generated content** before publishing
4. **Monitor API costs** regularly (OpenAI, Vercel, SendGrid)
5. **Test email templates** with your own address first
6. **Set rate limits** to avoid API bans
7. **Keep logs** of all deployments and campaigns
8. **Iterate based on metrics** — don't just deploy and forget

## Security & Privacy

- Never commit API keys to git
- Use environment variables or secure key storage (1Password)
- Review ToS before scraping (Reddit, HN allow it with respectful rate limits)
- Follow CAN-SPAM Act for email outreach (include unsubscribe)
- Don't scrape personal data without consent
- Use HTTPS for all deployments
- Rotate API tokens regularly

## Further Reading

- [OpenClaw Gateway Security](https://docs.openclaw.ai/gateway/security)
- [Skills Configuration](https://docs.openclaw.ai/tools/skills-config)
- Reddit API: https://www.reddit.com/dev/api
- HackerNews API: https://github.com/HackerNews/API
- Vercel API: https://vercel.com/docs/rest-api
- ProductHunt: https://www.producthunt.com (web scraping, no official API)

---

**Remember:** CEOClaw is a power tool for autonomous business building. Start small, test thoroughly, and always get approval before real-world actions. Building a business takes iteration — use the metrics to guide improvements.

## Usage

### Quick Start

Run the autonomous founder:

```bash
cd skills/ceoclaw/scripts
npm install
npm run ceoclaw
```

### Manual Tool Execution

Run individual tools:

```bash
# Market research
node scripts/market-research.js --topics "productivity,automation"

# Generate idea
node scripts/idea-generator.js --problem "developers need better logging"

# Build landing page
node scripts/landing-page-builder.js --idea "AI-powered code review tool"

# Deploy
node scripts/deploy.js --project ./landing-page

# Send marketing email
node scripts/email-outreach.js --recipients recipients.json
```

### Integration with OpenClaw Agents

CEOClaw can be invoked from OpenClaw agents:

```
openclaw agent --message "Launch a SaaS business targeting developers" --skill ceoclaw
```

## Configuration

Create `ceoclaw.config.json`:

```json
{
  "businessGoal": "reach-first-customer",
  "targetMarket": "developers",
  "budget": {
    "maxMonthlySpend": 100
  },
  "approval": {
    "requireForDeployments": true,
    "requireForEmails": true,
    "requireForPosts": true
  },
  "iterations": {
    "maxCycles": 10,
    "successCriteria": {
      "signups": 10,
      "revenue": 100
    }
  }
}
```

## Safety Features

1. **Approval Gates** - Human review required for sensitive actions
2. **Budget Limits** - Spending caps on API usage
3. **Audit Logging** - All actions logged to SQLite
4. **Dry Run Mode** - Test without executing real actions
5. **Rollback** - Undo deployments and retract emails (where possible)

## Examples

### Example 1: Launch Developer Tool

```bash
CEOCLAW_TARGET_MARKET="developers" \\
CEOCLAW_PROBLEM_SPACE="debugging" \\
npm run ceoclaw
```

### Example 2: Validate Idea Only

```bash
CEOCLAW_MODE="validate" \\
CEOCLAW_IDEA="API monitoring dashboard" \\
npm run ceoclaw
```

### Example 3: Marketing Campaign

```bash
CEOCLAW_MODE="marketing" \\
CEOCLAW_PRODUCT_URL="https://myproduct.com" \\
npm run ceoclaw
```

## Output

CEOClaw generates:

- **Business plan** - Strategy and execution steps
- **Landing page** - Production-ready HTML/React
- **Marketing content** - Blog posts, social media, emails
- **Metrics dashboard** - Real-time business KPIs
- **Experiment log** - All actions and results

All data stored in: `~/.ceoclaw/`

## Limitations

- **Small scale** - Designed for micro-businesses, not unicorns
- **Technical products** - Best for developer/tech audiences
- **Simple products** - Landing pages and basic web apps only
- **Manual review** - Complex decisions still need human judgment

## Ethics & Legal

**Use Responsibly:**
- Comply with CAN-SPAM Act (email)
- Respect platform Terms of Service
- Don't spam or harass users
- Be transparent about AI usage
- Follow GDPR/privacy regulations
- Get legal review for regulated industries

**The creators of CEOClaw are not responsible for misuse.**

## Troubleshooting

### "Deployment failed"
- Check Vercel token is valid
- Ensure project has valid `vercel.json`
- Verify no naming conflicts

### "Email sending failed"
- Confirm SMTP credentials
- Check recipient addresses are valid
- Verify not rate-limited

### "No market signals found"
- Try different search terms
- Check API rate limits
- Verify internet connection

## Learn More

- See `scripts/` for tool implementations
- See `references/` for detailed documentation
- See `examples/` for sample configurations

## Support

- GitHub Issues: https://github.com/openclaw/openclaw/issues
- Discord: https://discord.gg/clawd
- Docs: https://docs.openclaw.ai
