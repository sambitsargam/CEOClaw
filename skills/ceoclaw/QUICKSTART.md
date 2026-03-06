# CEOClaw Quick Start Guide

Get your autonomous AI founder running in 5 minutes.

## What is CEOClaw?

CEOClaw is an OpenClaw skill that teaches AI agents to autonomously start and operate internet businesses. Unlike traditional tools, CEOClaw is **pure guidance** - markdown documentation that shows the agent how to use existing CLIs, APIs, and tools to:

- Research market problems (Reddit, HackerNews, ProductHunt)
- Generate startup ideas (OpenAI GPT-4)
- Build landing pages (HTML generation)
- Deploy projects (Vercel API)
- Market products (Twitter, ProductHunt, email)
- Track metrics (Google Analytics, Plausible)

## Prerequisites

- OpenAI API key (for idea generation and content creation)
- (Optional) Vercel account for deployments
- (Optional) SMTP credentials for email outreach

## Step 1: Set Up Your Environment

Get your OpenAI API key from https://platform.openai.com/api-keys

```bash
export OPENAI_API_KEY="sk-your-key-here"
```

Optional services:

```bash
# Vercel (for deployments)
export CEOCLAW_VERCEL_TOKEN="your-vercel-token"

# SMTP (for email campaigns)
export CEOCLAW_SMTP_HOST="smtp.gmail.com"
export CEOCLAW_SMTP_USER="your-email@gmail.com"
export CEOCLAW_SMTP_PASS="your-app-password"

# Analytics (optional)
export CEOCLAW_GA_TRACKING_ID="G-XXXXXXXXXX"
export CEOCLAW_PLAUSIBLE_DOMAIN="your-domain.com"
```

## Step 2: Initialize Database

CEOClaw stores all business data (ideas, deployments, metrics) in SQLite.

Create `~/.ceoclaw/data.db`:

```bash
mkdir -p ~/.ceoclaw

sqlite3 ~/.ceoclaw/data.db <<'SQL'
CREATE TABLE IF NOT EXISTS market_signals (
  id TEXT PRIMARY KEY,
  source TEXT NOT NULL,
  problem TEXT NOT NULL,
  context TEXT,
  pain_level INTEGER,
  upvotes INTEGER DEFAULT 0,
  keywords TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS startup_ideas (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  tagline TEXT NOT NULL,
  problem TEXT NOT NULL,
  solution TEXT NOT NULL,
  target_market TEXT,
  demand_score INTEGER,
  feasibility_score INTEGER,
  moat_score INTEGER,
  status TEXT DEFAULT 'generated',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS landing_pages (
  id TEXT PRIMARY KEY,
  idea_id TEXT NOT NULL,
  url TEXT,
  html TEXT,
  copy TEXT,
  cta TEXT,
  deployed_at DATETIME,
  FOREIGN KEY(idea_id) REFERENCES startup_ideas(id)
);

CREATE TABLE IF NOT EXISTS deployments (
  id TEXT PRIMARY KEY,
  idea_id TEXT NOT NULL,
  url TEXT UNIQUE NOT NULL,
  platform TEXT DEFAULT 'vercel',
  status TEXT DEFAULT 'pending',
  deployed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY(idea_id) REFERENCES startup_ideas(id)
);

CREATE TABLE IF NOT EXISTS marketing_content (
  id TEXT PRIMARY KEY,
  idea_id TEXT NOT NULL,
  type TEXT NOT NULL,
  content TEXT NOT NULL,
  platform TEXT,
  posted_at DATETIME,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY(idea_id) REFERENCES startup_ideas(id)
);

CREATE TABLE IF NOT EXISTS metrics (
  id TEXT PRIMARY KEY,
  deployment_id TEXT NOT NULL,
  date DATE NOT NULL,
  visitors INTEGER DEFAULT 0,
  signups INTEGER DEFAULT 0,
  revenue REAL DEFAULT 0.0,
  FOREIGN KEY(deployment_id) REFERENCES deployments(id)
);

CREATE INDEX idx_market_signals_pain ON market_signals(pain_level DESC);
CREATE INDEX idx_startup_ideas_demand ON startup_ideas(demand_score DESC);
CREATE INDEX idx_deployments_status ON deployments(status);
CREATE INDEX idx_metrics_date ON metrics(date DESC);
SQL
```

Verify:

```bash
sqlite3 ~/.ceoclaw/data.db ".tables"
# Should show: deployments, landing_pages, market_signals, marketing_content, metrics, startup_ideas
```

## Step 3: Test Market Research

Let's manually test market research to understand the workflow:

```bash
# Research developer pain points on Reddit
curl -H "User-Agent: CEOClaw/1.0" \
  "https://www.reddit.com/r/programming/search.json?q=frustrated+OR+pain+point+OR+waste+time&limit=50&sort=top&t=month" \
  | jq -r '.data.children[] | select(.data.selftext != "") | 
    {
      title: .data.title,
      body: .data.selftext,
      score: .data.score,
      url: ("https://reddit.com" + .data.permalink)
    }' \
  | head -20
```

This shows real problems developers are discussing. The agent will use similar patterns to gather market data.

## Step 4: Ask the Agent to Start a Business

Now, instead of running manual commands, you simply ask your OpenClaw agent:

```bash
openclaw agent --message "I want to start an internet business targeting developers. Research market problems, generate 3 startup ideas, and create landing pages for the best one."
```

Or be more specific:

```bash
openclaw agent --message "Research productivity problems on HackerNews from the past month. Generate startup ideas that solve those problems. Build and deploy the most promising idea to Vercel."
```

The agent will:
1. Read [SKILL.md](./SKILL.md) to understand the 8-phase workflow
2. Follow [references/market-research.md](./references/market-research.md) to scrape data
3. Use [references/idea-generation.md](./references/idea-generation.md) to create ideas
4. Apply [references/landing-pages.md](./references/landing-pages.md) to build HTML
5. Deploy using [references/deployment.md](./references/deployment.md)
6. Track progress in `~/.ceoclaw/data.db`

## Step 5: Review What the Agent Created

Check the database to see generated ideas:

```bash
sqlite3 ~/.ceoclaw/data.db "SELECT name, tagline, demand_score, status FROM startup_ideas ORDER BY demand_score DESC LIMIT 3;"
```

View deployed projects:

```bash
sqlite3 ~/.ceoclaw/data.db "SELECT s.name, d.url, d.deployed_at FROM deployments d JOIN startup_ideas s ON d.idea_id = s.id WHERE d.status = 'active' ORDER BY d.deployed_at DESC;"
```

Check marketing content:

```bash
sqlite3 ~/.ceoclaw/data.db "SELECT s.name, m.type, m.platform, m.posted_at FROM marketing_content m JOIN startup_ideas s ON m.idea_id = s.id ORDER BY m.created_at DESC LIMIT 5;"
```

## Step 6: Run the Full Business Workflow (Advanced)

For a complete 8-phase autonomous business:

```bash
# Copy the business workflow script
curl -s https://raw.githubusercontent.com/openclaw/openclaw/main/skills/ceoclaw/references/business-workflow.md \
  | sed -n '/```bash/,/```/p' | sed '1d;$d' > /tmp/run-business.sh

chmod +x /tmp/run-business.sh

# Run with safety checks (dry run)
DRY_RUN=true /tmp/run-business.sh

# Run for real (⚠️ WARNING: Will spend money)
/tmp/run-business.sh
```

This script orchestrates all 8 phases with approval gates.

## Step 7: Monitor and Iterate

Ask the agent to check progress:

```bash
openclaw agent --message "Show me metrics for all my deployed projects. Which ones are getting traction?"
```

Or manually:

```bash
sqlite3 ~/.ceoclaw/data.db "
SELECT 
  s.name,
  d.url,
  SUM(m.visitors) as total_visitors,
  SUM(m.signups) as total_signups,
  SUM(m.revenue) as total_revenue
FROM deployments d
JOIN startup_ideas s ON d.idea_id = s.id
LEFT JOIN metrics m ON m.deployment_id = d.id
WHERE d.status = 'active'
GROUP BY s.id
ORDER BY total_visitors DESC;
"
```

## Example Agent Prompts

Here are effective ways to interact with CEOClaw:

**Research Only:**
```bash
openclaw agent --message "Research SaaS pain points on Reddit's r/SaaS and r/entrepreneur from the past 2 weeks. Store findings in the database."
```

**Idea Generation:**
```bash
openclaw agent --message "Generate 5 startup ideas for remote workers. Focus on ideas with low upfront cost (<$100) and clear monetization."
```

**Full Launch:**
```bash
openclaw agent --message "Launch a business: (1) Research developer tools pain points, (2) Generate 3 ideas, (3) Build landing page for best idea, (4) Deploy to Vercel, (5) Post to ProductHunt and Twitter. Ask for approval before deployments and posts."
```

**Marketing Focus:**
```bash
openclaw agent --message "For my deployed project at https://myapp.vercel.app, create a Twitter thread, blog post, and email outreach campaign to developers."
```

**Metrics Review:**
```bash
openclaw agent --message "Analyze metrics for all my projects. Which should I double down on? Which should I shut down?"
```

## Understanding the Skill Structure

CEOClaw consists of:

1. **[SKILL.md](./SKILL.md)**: Main skill definition - read this to understand the 8-phase workflow
2. **[references/](./references/)**: Detailed guides for each phase:
   - `market-research.md`: Scraping techniques for Reddit, HN, PH
   - `idea-generation.md`: GPT-4 prompting for ideation
   - `landing-pages.md`: HTML generation patterns
   - `deployment.md`: Vercel API workflows
   - `marketing.md`: Content creation and distribution
   - `email-outreach.md`: SMTP campaigns
   - `metrics.md`: Analytics tracking
   - `business-workflow.md`: Complete orchestration script

3. **Database**: SQLite at `~/.ceoclaw/data.db` stores everything

The agent reads these markdown files and executes the workflows using OpenClaw's built-in tools (`run_in_terminal`, `fetch_webpage`, `create_file`, etc.).

## Safety Best Practices

⚠️ **Before enabling full automation, review:**

1. **[references/safety-guidelines.md](./references/safety-guidelines.md)**: Complete safety checklist
2. **Cost limits**: Set `CEOCLAW_MAX_MONTHLY_SPEND` to prevent overspending
3. **Approval gates**: Always require approval for deployments, emails, and social posts initially
4. **Test in dry-run mode**: Use `DRY_RUN=true` for the business workflow script
5. **Monitor closely**: Check database and deployed sites daily

## Troubleshooting

**"OPENAI_API_KEY not set":**
```bash
export OPENAI_API_KEY="sk-..."
```

**"Database locked":**
```bash
# Another process is using it. Check:
fuser ~/.ceoclaw/data.db
```

**"Vercel deployment failed":**
```bash
# Test Vercel token manually:
curl -H "Authorization: Bearer $CEOCLAW_VERCEL_TOKEN" https://api.vercel.com/v9/projects
```

**"Agent doesn't invoke CEOClaw skill":**

Make sure your prompt includes business/startup/entrepreneurship keywords. Try:
```bash
openclaw agent --message "Use the CEOClaw skill to research market opportunities"
```

## Next Steps

1. **Read [SKILL.md](./SKILL.md)**: Understand the complete 8-phase workflow
2. **Browse reference guides**: See [references/](./references/) for detailed patterns
3. **Start small**: Begin with market research only, then expand
4. **Review safety**: Read [references/safety-guidelines.md](./references/safety-guidelines.md)
5. **Monitor results**: Check database and track metrics daily
6. **Iterate**: Refine prompts based on what the agent creates

## Advanced Usage

**Custom GPT-4 Instructions:**

Add to your OpenClaw agent config:

```json
{
  "systemPrompt": "You are an autonomous AI founder. Be bold but cost-conscious. Prioritize demand validation before building. Fail fast on low-traction ideas."
}
```

**Multi-Business Management:**

Track multiple projects:

```bash
sqlite3 ~/.ceoclaw/data.db "
SELECT 
  s.name,
  s.status,
  COUNT(DISTINCT d.id) as deployments,
  MAX(m.visitors) as peak_visitors
FROM startup_ideas s
LEFT JOIN deployments d ON d.idea_id = s.id
LEFT JOIN metrics m ON m.deployment_id = d.id
GROUP BY s.id
ORDER BY s.created_at DESC;
"
```

**Scheduled Iteration:**

Run nightly via cron:

```bash
# Add to crontab: crontab -e
0 2 * * * /tmp/run-business.sh 2>&1 | tee -a ~/.ceoclaw/business.log
```

The agent will autonomously research, iterate, and optimize your deployed businesses.

## Support

- GitHub: https://github.com/openclaw/openclaw/issues
- Discord: https://discord.gg/clawd
- Docs: https://docs.openclaw.ai/skills/ceoclaw

## Example Run

```bash
$ npm run ceoclaw

╔═══════════════════════════════════════════════╗
║                                               ║
║   👔 CEOClaw - Autonomous AI Founder          ║
║                                               ║
║   Starting business execution...              ║
║                                               ║
╚═══════════════════════════════════════════════╝

ℹ Runner Configuration loaded
  Business Goal: reach-first-customer
  Target Market: developers
  Max Cycles: 5
⚠ Runner ⚠️  AUTO-DEPLOY ENABLED - Will deploy to production automatically
ℹ Runner Starting in 3 seconds... (Ctrl+C to cancel)
ℹ 👔 CEOClaw 🚀 Starting CEOClaw Business Execution Loop
  Goal: reach-first-customer
  Target Market: developers
  Max Cycles: 5

━━━ CYCLE 1/5 ━━━

━━━ MARKET DISCOVERY ━━━

ℹ MarketResearch Starting market research for topics: developers
ℹ MarketResearch Scraped 12 signals from Reddit
ℹ MarketResearch Scraped 8 signals from HackerNews
ℹ MarketResearch Scraped 5 signals from ProductHunt
✓ MarketResearch Found 25 market signals
✓ 👔 CEOClaw Discovered 25 market signals
  Top Keywords: debugging, logs, testing, deploy, cicd
  Avg Pain Level: 7.2/10

━━━ IDEA GENERATION ━━━

ℹ IdeaGenerator Generating 3 startup ideas from 25 market signals
✓ IdeaGenerator Generated 3 startup ideas
✓ 👔 CEOClaw Generated 3 ideas
ℹ 👔 CEOClaw 
🎯 Selected: DevLogger
   Beautiful, AI-powered logging for developers
  Demand Score: 87/100

...
```

## Pro Tips

1. **Start Small**: Use low max_cycles (3-5) initially
2. **Review Everything**: Keep approval gates enabled
3. **Monitor Costs**: Track OpenAI API usage
4. **Iterate**: Adjust target market based on results
5. **Be Patient**: Real business building takes time

Happy founding! 🚀
