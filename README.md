# 👔 CEOClaw — Autonomous AI Founder

<p align="center">
  <strong>Launch and operate internet businesses autonomously using AI</strong>
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw"><img src="https://img.shields.io/badge/Built_with-OpenClaw-blue?style=for-the-badge" alt="Built with OpenClaw"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="MIT License"></a>
</p>

**CEOClaw** is an OpenClaw skill that teaches AI agents to autonomously start and operate small internet businesses. It's pure markdown guidance—no source code—just documentation that shows the agent how to use existing CLIs, APIs, and tools to execute the complete entrepreneurship lifecycle.

## 🎯 What CEOClaw Does

CEOClaw guides your AI agent through an **8-phase business workflow**:

1. **Market Research** — Scrape Reddit, HackerNews, ProductHunt for real user problems
2. **Idea Generation** — Generate validated startup ideas with demand scoring
3. **Problem-Solution Validation** — Analyze market signals and assess viability
4. **Landing Page Creation** — Build production-ready HTML pages with Tailwind CSS
5. **Automated Deployment** — Deploy to Vercel via API or CLI
6. **Marketing Campaigns** — Generate Twitter threads, blog posts, ProductHunt launches
7. **Email Outreach** — SMTP campaigns with personalization and drip sequences
8. **Metrics & Analytics** — Track visitors, signups, revenue via Google Analytics/Plausible

## ✨ Key Features

- **🎨 Pure Markdown Skill** — No code to maintain; the agent reads documentation and executes using OpenClaw's built-in tools
- **🤖 Any LLM** — Works with Anthropic Claude, OpenAI GPT-4, Google Gemini, or any model configured in OpenClaw
- **🔌 Real Integrations** — Actual APIs for Vercel deployment, Reddit/HN scraping, SMTP email, analytics tracking
- **🛡️ Safety First** — Approval gates, cost limits, dry-run mode prevent accidents
- **💾 SQLite Persistence** — All business data (ideas, deployments, metrics) stored locally
- **♻️ Complete Automation** — From market research to revenue tracking, fully autonomous operation
- **📊 Business Intelligence** — Built-in analytics and reporting on what's working

## 🚀 Quick Start

### Prerequisites

- OpenClaw installed and configured with any LLM provider
- (Optional) Vercel account for deployments
- (Optional) SMTP credentials for email campaigns

### Installation

CEOClaw is already included in this repository. Install it to your OpenClaw workspace:

```bash
# Copy CEOClaw to OpenClaw workspace
mkdir -p ~/.openclaw/workspace/skills
cp -r skills/ceoclaw ~/.openclaw/workspace/skills/

# Verify installation
openclaw skills list | grep ceoclaw
# Should show: ✓ ready | 👔 ceoclaw | Autonomous AI founder...
```

### Initialize Database

CEOClaw stores all business data in SQLite:

```bash
# Create database directory
mkdir -p ~/.ceoclaw

# Initialize CEOClaw database schema
sqlite3 ~/.ceoclaw/data.db < skills/ceoclaw/references/schema.sql

# Or let the agent initialize it:
openclaw agent --session-id main --message "Initialize CEOClaw database with the schema from the SKILL.md file"
```

### Your First Business

```bash
# Start OpenClaw gateway
openclaw gateway run

# In another terminal, launch a business:
openclaw agent --session-id main --message "Research developer productivity problems on Reddit and HackerNews from the past month. Generate 3 startup ideas and create a landing page for the best one."

# Check what was created:
sqlite3 ~/.ceoclaw/data.db "SELECT name, tagline, demand_score FROM startup_ideas ORDER BY demand_score DESC;"
```

## 📖 Documentation

- **[QUICKSTART.md](skills/ceoclaw/QUICKSTART.md)** — 5-minute setup guide
- **[SKILL.md](skills/ceoclaw/SKILL.md)** — Complete skill definition and workflow
- **[IMPLEMENTATION.md](skills/ceoclaw/IMPLEMENTATION.md)** — Technical architecture overview

### Reference Guides

- **[Market Research](skills/ceoclaw/references/market-research.md)** — Reddit, HackerNews, ProductHunt scraping patterns
- **[Idea Generation](skills/ceoclaw/references/idea-generation.md)** — LLM prompting strategies for startup ideation
- **[Landing Pages](skills/ceoclaw/references/landing-pages.md)** — HTML generation with Tailwind CSS
- **[Deployment](skills/ceoclaw/references/deployment.md)** — Vercel API and CLI workflows
- **[Marketing](skills/ceoclaw/references/marketing.md)** — Content creation and distribution
- **[Email Outreach](skills/ceoclaw/references/email-outreach.md)** — SMTP campaign setup
- **[Metrics](skills/ceoclaw/references/metrics.md)** — Analytics tracking and reporting
- **[Business Workflow](skills/ceoclaw/references/business-workflow.md)** — Complete orchestration script
- **[API Integrations](skills/ceoclaw/references/api-integrations.md)** — API authentication guides
- **[Safety Guidelines](skills/ceoclaw/references/safety-guidelines.md)** — Ethics and compliance

## 💡 Example Use Cases

### Research & Ideation

```bash
openclaw agent --session-id main --message "Research SaaS pain points on Reddit's r/SaaS and r/entrepreneur from the past 2 weeks. Store findings in the database."
```

### Full Business Launch

```bash
openclaw agent --session-id main --message "Launch a business targeting remote workers: (1) Research problems, (2) Generate 3 ideas, (3) Build landing page for best idea, (4) Deploy to Vercel, (5) Create marketing content. Ask for approval before deployments."
```

### Marketing Focus

```bash
openclaw agent --session-id main --message "For my deployed project at https://myapp.vercel.app, create a Twitter thread, blog post, and email campaign targeting developers."
```

### Analytics & Iteration

```bash
openclaw agent --session-id main --message "Analyze metrics for all my deployed projects. Which ones are getting traction? Which should I shut down?"
```

## 🔧 Configuration

### Optional Environment Variables

```bash
# Vercel Deployment
export CEOCLAW_VERCEL_TOKEN="..."                 # Vercel API token

# Email Outreach
export CEOCLAW_SMTP_HOST="smtp.gmail.com"         # SMTP server
export CEOCLAW_SMTP_PORT="587"                    # SMTP port
export CEOCLAW_SMTP_USER="you@example.com"        # Email account
export CEOCLAW_SMTP_PASS="..."                    # App password

# Analytics (Optional)
export CEOCLAW_GA_TRACKING_ID="G-..."             # Google Analytics 4
export CEOCLAW_PLAUSIBLE_DOMAIN="yourdomain.com"  # Plausible domain

# Safety Limits
export CEOCLAW_MAX_MONTHLY_SPEND="50"             # Maximum monthly spend in USD
```

## 🛡️ Safety & Best Practices

⚠️ **CEOClaw performs real-world actions with real consequences:**

- ✅ **Always test in dry-run mode first**
- ✅ **Use approval gates** for deployments, emails, and social posts
- ✅ **Set cost limits** to prevent overspending
- ✅ **Review generated content** before publishing
- ✅ **Monitor database and deployed sites** regularly
- ✅ **Read [Safety Guidelines](skills/ceoclaw/references/safety-guidelines.md)**

## 🏗️ Architecture

CEOClaw is a **markdown-based skill** following OpenClaw conventions:

```
skills/ceoclaw/
├── SKILL.md                    # Main skill definition (YAML frontmatter + docs)
├── README.md                   # This file
├── QUICKSTART.md              # Quick setup guide
├── IMPLEMENTATION.md          # Technical overview
├── examples/
│   └── ceoclaw.config.json   # Example configuration
└── references/                # Detailed implementation guides
    ├── market-research.md
    ├── idea-generation.md
    ├── landing-pages.md
    ├── deployment.md
    ├── marketing.md
    ├── email-outreach.md
    ├── metrics.md
    ├── business-workflow.md
    ├── api-integrations.md
    └── safety-guidelines.md
```

### How It Works

1. **Agent reads SKILL.md** — Understands capabilities and 8-phase workflow
2. **Follows reference guides** — Executes bash scripts, curl commands, node one-liners
3. **Uses OpenClaw tools** — `run_in_terminal`, `fetch_webpage`, `create_file`, etc.
4. **Stores in SQLite** — All data persists in `~/.ceoclaw/data.db`
5. **Reports progress** — Updates stored in database for tracking

No custom code required—the agent interprets markdown and executes using existing tools.

## 📊 Database Schema

CEOClaw uses SQLite for persistence:

- **market_signals** — Problems scraped from Reddit/HN/PH
- **startup_ideas** — Generated ideas with demand scores
- **landing_pages** — HTML content and copy
- **deployments** — Live URLs and deployment status
- **marketing_content** — Tweets, posts, email campaigns
- **metrics** — Daily visitors, signups, revenue

Query your data:

```bash
# Top ideas by demand score
sqlite3 ~/.ceoclaw/data.db "
SELECT name, tagline, demand_score, feasibility_score
FROM startup_ideas
ORDER BY demand_score DESC
LIMIT 5;
"

# Deployment performance
sqlite3 ~/.ceoclaw/data.db "
SELECT s.name, d.url, SUM(m.visitors) as total_visitors
FROM deployments d
JOIN startup_ideas s ON d.idea_id = s.id
LEFT JOIN metrics m ON m.deployment_id = d.id
GROUP BY d.id
ORDER BY total_visitors DESC;
"
```

## 🤝 Contributing

CEOClaw is part of the OpenClaw ecosystem. Contributions welcome!

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to the branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

## 📜 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **OpenClaw Team** — For creating the AI assistant framework that makes CEOClaw possible
- **AI Model Providers** — Anthropic, OpenAI, Google for powerful LLMs
- **Open Source Community** — For the tools and APIs that power autonomous business operations

## 🔗 Related Projects

- **[OpenClaw](https://github.com/openclaw/openclaw)** — Personal AI assistant framework (required to run CEOClaw)
- **[OpenClaw Docs](https://docs.openclaw.ai)** — Complete OpenClaw documentation
- **[OpenClaw Skills](https://docs.openclaw.ai/tools/skills)** — Learn how to create your own skills

## 📞 Support

- 💬 **Discord**: [Join OpenClaw Discord](https://discord.gg/clawd)
- 📖 **Documentation**: [docs.openclaw.ai](https://docs.openclaw.ai)
- 🐛 **Issues**: [GitHub Issues](https://github.com/openclaw/openclaw/issues)
- 🌐 **Website**: [openclaw.ai](https://openclaw.ai)

---

<p align="center">
  <strong>Built with OpenClaw • Launch businesses autonomously with AI</strong>
</p>
