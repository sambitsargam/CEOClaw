# CEOClaw — Autonomous AI Founder

OpenClaw skill that guides you through launching and operating small internet businesses autonomously.

## What is CEOClaw?

CEOClaw is a **markdown-based skill** that teaches OpenClaw's AI agent how to execute the complete lifecycle of starting an internet business:

1. **Market Research** — Scrape Reddit, HackerNews, ProductHunt for problems
2. **Idea Generation** — Synthesize validated startup ideas with GPT-4
3. **MVP Building** — Generate responsive landing pages
4. **Deployment** — Deploy to Vercel with custom domains
5. **Marketing** — Create and distribute content across channels
6. **User Outreach** — Send personalized emails to early adopters
7. **Metrics Tracking** — Monitor traffic, signups, conversions
8. **Iteration** — Improve based on data and feedback

## How It Works

Unlike traditional code implementations, CEOClaw provides **guidance documentation** that teaches the OpenClaw agent how to:
- Use existing CLIs and APIs (curl, node, vercel, etc.)
- Write bash scripts for automation
- Call GPT-4 for content generation
- Store data in SQLite databases
- Execute real-world business tasks

The agent reads the skill's markdown files and executes tasks using its built-in tools.

## Project Structure

```
skills/ceoclaw/
├── SKILL.md                    # Main skill definition
├── README.md                   # This file
├── QUICKSTART.md               # 5-minute setup guide
├── IMPLEMENTATION.md           # Complete summary
├── examples/
│   └── ceoclaw.config.json    # Example configuration
└── references/
    ├── market-research.md      # Reddit/HN/PH scraping guide
    ├── idea-generation.md      # GPT-4 ideation prompts
    ├── landing-pages.md        # HTML generation guide
    ├── deployment.md           # Vercel deployment workflows
    ├── marketing.md            # Content generation & distribution
    ├── email-outreach.md       # SMTP campaign setup
    ├── metrics.md              # Analytics tracking
    ├── business-workflow.md    # Complete 8-phase orchestration
    ├── api-integrations.md     # API authentication guides
    └── safety-guidelines.md    # Ethics and compliance
```

## Prerequisites

### Environment Variables

```bash
# Essential
export OPENAI_API_KEY="sk-..."                    # GPT-4 for content generation

# Deployment (choose one)
export CEOCLAW_VERCEL_TOKEN="..."                 # Vercel API token
# OR: npm i -g vercel && vercel login

# Email (choose one)
export CEOCLAW_SMTP_HOST="smtp.gmail.com"         # SMTP server
export CEOCLAW_SMTP_PORT="587"                    # SMTP port
export CEOCLAW_SMTP_USER="you@example.com"        # Email account
export CEOCLAW_SMTP_PASS="..."                    # App password
# OR:
export CEOCLAW_SENDGRID_KEY="SG...."              # SendGrid API key

# Optional
export CEOCLAW_ANALYTICS_ID="G-..."               # Google Analytics 4
export CEOCLAW_PLAUSIBLE_DOMAIN="yourdomain.com"  # Plausible domain
export CEOCLAW_REDDIT_CLIENT_ID="..."             # Reddit API
export CEOCLAW_REDDIT_SECRET="..."                # Reddit API secret
```

### Required Tools

- `curl` or `node` — HTTP requests
- `node` ≥22 — Script execution
- `jq` — JSON parsing
- `sqlite3` — Database management (optional)
- `vercel` CLI — Deployments (optional)

## Quick Start

### 1. Initialize Database

```bash
mkdir -p ~/.openclaw/ceoclaw
cd ~/.openclaw/ceoclaw

sqlite3 ceoclaw.db < /path/to/skills/ceoclaw/schema.sql
```

### 2. Ask OpenClaw to Launch a Business

```bash
# Via OpenClaw CLI
openclaw agent --message "Use CEOClaw to launch a business in the developer tools space. Start with market research, generate ideas, and build a landing page. Run in approval mode."

# Or in chat
"Use CEOClaw skill to find SaaS problems on Reddit and generate 3 startup ideas"
```

### 3. The Agent Will:

1. Read `SKILL.md` and reference guides
2. Execute shell commands to scrape Reddit/HN
3. Call GPT-4 API for idea generation
4. Generate HTML landing page
5. Deploy to Vercel (with your approval)
6. Set up analytics tracking
7. Create marketing content
8. Track metrics in SQLite database

## Usage Examples

### Example 1: Market Research Only

```
"Use CEOClaw to research SaaS problems on Reddit. 
Focus on /r/SaaS and /r/startups from the last month. 
Store top 10 problems in the database."
```

### Example 2: Full Launch with Approval Gates

```
"Launch a new business using CEOClaw:
1. Research problems in the productivity space
2. Generate and score 5 ideas
3. Pick the best one and build a landing page
4. Deploy to Vercel
5. Create a ProductHunt launch post

Ask for approval before deployments and posts."
```

### Example 3: Iteration on Existing Business

```
"I have a live product at https://myproduct.vercel.app
Use CEOClaw to:
- Fetch current metrics from Google Analytics
- Analyze traffic sources
- Generate 3 A/B test variations for the headline
- Create social media posts for top-performing channels"
```

## Documentation

All detailed workflows live in `references/`:

- **[market-research.md](references/market-research.md)** — Reddit, HackerNews, ProductHunt scraping
- **[idea-generation.md](references/idea-generation.md)** — GPT-4 prompts for startup ideation
- **[landing-pages.md](references/landing-pages.md)** — HTML/React generation
- **[deployment.md](references/deployment.md)** — Vercel API + CLI deployment
- **[marketing.md](references/marketing.md)** — Content creation & distribution
- **[email-outreach.md](references/email-outreach.md)** — SMTP campaigns
- **[metrics.md](references/metrics.md)** — Analytics integration
- **[business-workflow.md](references/business-workflow.md)** — Complete orchestration
- **[api-integrations.md](references/api-integrations.md)** — API setup guides
- **[safety-guidelines.md](references/safety-guidelines.md)** — Ethics & compliance

## Safety & Best Practices

### Always:
✅ Run in dry-run mode first to preview actions  
✅ Get approval before deployments, emails, and posts  
✅ Review all generated content before publishing  
✅ Start with test/staging environments  
✅ Monitor API costs and rate limits  
✅ Follow CAN-SPAM Act for email (include unsubscribe)  
✅ Respect platform ToS (Reddit, HN rate limits)  

### Never:
❌ Deploy without testing locally first  
❌ Send emails to purchased lists  
❌ Scrape personal data without consent  
❌ Spam communities with promotional content  
❌ Ignore platform-specific posting rules  
❌ Use aggressive or deceptive marketing  

## How the Agent Uses This Skill

When you invoke CEOClaw, the OpenClaw agent:

1. **Reads SKILL.md** — Understands capabilities and workflow
2. **Consults references/** — Gets detailed instructions for each phase
3. **Executes commands** — Uses bash, curl, node, APIs
4. **Stores data** — SQLite database for persistence
5. **Reports progress** — Updates you at each phase
6. **Requests approval** — Before destructive actions
7. **Iterates** — Based on metrics and feedback

The agent has access to:
- `run_in_terminal` — Execute shell commands
- `fetch_webpage` — HTTP requests
- `create_file` — Generate scripts/content
- `read_file` — Access reference documentation

## Database Schema

Located at `~/.openclaw/ceoclaw/ceoclaw.db`:

```sql
-- Market signals from research
CREATE TABLE market_signals (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  source TEXT NOT NULL,
  problem TEXT NOT NULL,
  pain_level INTEGER,
  keywords TEXT,
  url TEXT,
  engagement INTEGER,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Generated business ideas
CREATE TABLE ideas (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  description TEXT,
  target_market TEXT,
  solution TEXT,
  score INTEGER,
  status TEXT DEFAULT 'draft',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Live deployments
CREATE TABLE deployments (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  idea_id INTEGER,
  url TEXT NOT NULL,
  platform TEXT,
  status TEXT,
  deployed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (idea_id) REFERENCES ideas(id)
);

-- Performance metrics
CREATE TABLE metrics (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  deployment_id INTEGER,
  date DATE,
  visits INTEGER DEFAULT 0,
  signups INTEGER DEFAULT 0,
  conversions INTEGER DEFAULT 0,
  revenue REAL DEFAULT 0,
  FOREIGN KEY (deployment_id) REFERENCES deployments(id)
);
```

## API Rate Limits

- **Reddit**: 60 req/min (unauthenticated), 100 req/min (OAuth)
- **HackerNews**: No official limit (be respectful, ~1 req/sec)
- **OpenAI GPT-4**: Depends on tier (check dashboard)
- **Vercel**: 100 deployments/day (Hobby), unlimited (Pro)
- **Google Analytics**: 2000 requests/day (free tier)
- **SendGrid**: 100 emails/day (free), higher with paid plans

## Troubleshooting

### Common Issues

**"OPENAI_API_KEY not set"**
```bash
export OPENAI_API_KEY="sk-..."
# Or add to ~/.zshrc / ~/.bashrc
```

**"Database locked"**
```bash
# Close other sqlite3 connections
fuser ~/.openclaw/ceoclaw/ceoclaw.db
kill <pid>
```

**"Vercel deployment failed"**
```bash
# Verify token
curl -H "Authorization: Bearer $CEOCLAW_VERCEL_TOKEN" \
  https://api.vercel.com/v2/user
```

**"Reddit 429 Too Many Requests"**
```bash
# Add delays between requests
sleep 2  # Between each request
# Or authenticate for higher limits
```

**"Emails going to spam"**
- Use authenticated domain (not gmail.com)
- Set up SPF, DKIM, DMARC records
- Avoid spam trigger words
- Include plain text version

### Getting Help

1. Check reference guides in `references/`
2. Review example scripts in documentation
3. Test components individually before full workflow
4. Review [safety-guidelines.md](references/safety-guidelines.md) for best practices

## Roadmap

Future enhancements:
- [ ] A/B testing automation
- [ ] Payment integration (Stripe)
- [ ] Customer support chatbot
- [ ] Multi-platform deployment (Netlify, Cloudflare)
- [ ] Advanced analytics dashboards
- [ ] Social media scheduling
- [ ] SEO optimization tools

## Contributing

Found a bug or have a feature request?  
Open an issue: https://github.com/openclaw/openclaw/issues

## License

Same as OpenClaw parent project (MIT).

## Credits

CEOClaw is an OpenClaw skill that demonstrates autonomous task execution for business operations. It uses:
- OpenAI GPT-4 for content generation
- Vercel for hosting
- Reddit/HN/ProductHunt for market research
- Standard web technologies (HTML, CSS, JavaScript)

---

**Remember**: CEOClaw is a powerful tool. Start small, test thoroughly, and always get approval before real-world actions. Building a business takes iteration — use the metrics to guide improvements.
