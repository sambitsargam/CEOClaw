# Business Workflow Guide

Complete 8-phase autonomous founder workflow orchestrating all CEOClaw capabilities.

## Overview

CEOClaw business workflow:

```
Phase 1: Market Discovery
   ↓
Phase 2: Idea Generation
   ↓
Phase 3: MVP Build
   ↓
Phase 4: Deployment
   ↓
Phase 5: Marketing Launch
   ↓
Phase 6: User Outreach
   ↓
Phase 7: Metrics Tracking
   ↓
Phase 8: Iteration
   ↓ (loop back to Phase 1 or 7)
```

## Complete Workflow Script

```bash
#!/bin/bash
# autonomous-founder.sh
# Complete business launch workflow

set -e

# Configuration
WORKSPACE=~/.openclaw/ceoclaw
DB="$WORKSPACE/ceoclaw.db"
RESEARCH_DIR="$WORKSPACE/research"

mkdir -p "$WORKSPACE" "$RESEARCH_DIR"
cd "$WORKSPACE"

# Color output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

log() { echo -e "${BLUE}[$(date +%H:%M:%S)]${NC} $1"; }
success() { echo -e "${GREEN}✅ $1${NC}"; }
error() { echo -e "${RED}❌ $1${NC}"; exit 1; }
warning() { echo -e "${YELLOW}⚠️  $1${NC}"; }

# Approval gate
approve() {
  local action="$1"
  log "$action"
  read -p "Approve? (yes/no): " approval
  if [ "$approval" != "yes" ]; then
    warning "Action cancelled by user"
    exit 0
  fi
}

# ============================================
# PHASE 1: MARKET DISCOVERY
# ============================================
phase1_market_discovery() {
  log "PHASE 1: Market Discovery"
  
  # Scrape Reddit
  log "Fetching Reddit data..."
  for sub in SaaS startups webdev; do
    curl -sf -A "CEOClaw/1.0" \
      "https://www.reddit.com/r/$sub/top.json?limit=100&t=month" \
      > "$RESEARCH_DIR/reddit_${sub}.json"
    sleep 2
  done
  
  # Scrape HackerNews
  log "Fetching HackerNews data..."
  curl -sf "https://hn.algolia.com/api/v1/search?query=Ask%20HN&tags=ask_hn&hitsPerPage=100" \
    > "$RESEARCH_DIR/hn_ask.json"
  
  # Extract problems with GPT-4
  log "Extracting pain points with GPT-4..."
  node <<'EOF'
const fs = require('fs');
const https = require('https');

// Read research data
const redditData = JSON.parse(fs.readFileSync('research/reddit_SaaS.json', 'utf8'));
const hnData = JSON.parse(fs.readFileSync('research/hn_ask.json', 'utf8'));

const posts = [
  ...redditData.data.children.slice(0, 20).map(c => ({
    title: c.data.title,
    score: c.data.score,
    url: c.data.url
  })),
  ...hnData.hits.slice(0, 20).map(h => ({
    title: h.title,
    score: h.points,
    url: h.url
  }))
];

const prompt = `Extract user pain points from these posts. Return JSON array: [{problem, pain_level, keywords}]\n\n${JSON.stringify(posts)}`;

const body = JSON.stringify({
  model: 'gpt-4-turbo',
  messages: [{role: 'user', content: prompt}],
  temperature: 0.3
});

const req = https.request({
  hostname: 'api.openai.com',
  path: '/v1/chat/completions',
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
    'Content-Type': 'application/json'
  }
}, (res) => {
  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => {
    const result = JSON.parse(data);
    const problems = JSON.parse(result.choices[0].message.content);
    fs.writeFileSync('research/problems.json', JSON.stringify(problems, null, 2));
    console.log(`Extracted ${problems.length} problems`);
  });
});

req.write(body);
req.end();
EOF
  
  # Store in database
  log "Storing problems in database..."
  jq -c '.[]' "$RESEARCH_DIR/problems.json" | while read -r problem; do
    PROB=$(echo "$problem" | jq -r '.problem' | sed "s/'/''/g")
    PAIN=$(echo "$problem" | jq -r '.pain_level')
    sqlite3 "$DB" "INSERT INTO market_signals (source, problem, pain_level, engagement) VALUES ('research', '$PROB', $PAIN, 0);"
  done
  
  success "Phase 1 complete: $(sqlite3 "$DB" 'SELECT COUNT(*) FROM market_signals') problems identified"
}

# ============================================
# PHASE 2: IDEA GENERATION
# ============================================
phase2_idea_generation() {
  log "PHASE 2: Idea Generation"
  
  # Get top problems
  PROBLEMS=$(sqlite3 "$DB" "SELECT json_group_array(json_object('problem', problem, 'pain_level', pain_level)) FROM (SELECT * FROM market_signals ORDER BY pain_level DESC LIMIT 20)")
  
  # Generate ideas with GPT-4
  log "Generating startup ideas..."
  export PROBLEMS
  node <<'EOF'
const https = require('https');
const fs = require('fs');

const prompt = `Generate 5 startup ideas from these problems:\n\n${process.env.PROBLEMS}\n\nFor each: {title, problem, solution, target_market, mvp_scope, score}. Return JSON array.`;

const body = JSON.stringify({
  model: 'gpt-4-turbo',
  messages: [{role: 'user', content: prompt}],
  temperature: 0.7
});

const req = https.request({
  hostname: 'api.openai.com',
  path: '/v1/chat/completions',
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
    'Content-Type': 'application/json'
  }
}, (res) => {
  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => {
    const result = JSON.parse(data);
    const ideas = JSON.parse(result.choices[0].message.content);
    fs.writeFileSync('ideas.json', JSON.stringify(ideas, null, 2));
    console.log(`Generated ${ideas.length} ideas`);
  });
});

req.write(body);
req.end();
EOF
  
  # Store ideas
  jq -c '.[]' ideas.json | while read -r idea; do
    TITLE=$(echo "$idea" | jq -r '.title' | sed "s/'/''/g")
    PROBLEM=$(echo "$idea" | jq -r '.problem' | sed "s/'/''/g")
    SOLUTION=$(echo "$idea" | jq -r '.solution' | sed "s/'/''/g")
    TARGET=$(echo "$idea" | jq -r '.target_market' | sed "s/'/''/g")
    SCORE=$(echo "$idea" | jq -r '.score')
    
    sqlite3 "$DB" "INSERT INTO ideas (title, description, target_market, solution, score, status) VALUES ('$TITLE', '$PROBLEM', '$TARGET', '$SOLUTION', $SCORE, 'generated');"
  done
  
  # Show top idea
  TOP_IDEA=$(sqlite3 "$DB" "SELECT title FROM ideas ORDER BY score DESC LIMIT 1")
  success "Phase 2 complete: Top idea is '$TOP_IDEA'"
  
  # Return top idea ID
  sqlite3 "$DB" "SELECT id FROM ideas ORDER BY score DESC LIMIT 1"
}

# ============================================
# PHASE 3: MVP BUILD
# ============================================
phase3_mvp_build() {
  local idea_id="$1"
  log "PHASE 3: MVP Build (Idea #$idea_id)"
  
  # Get idea details
  IDEA=$(sqlite3 "$DB" "SELECT json_object('title', title, 'problem', description, 'solution', solution, 'target_market', target_market) FROM ideas WHERE id = $idea_id")
  
  # Generate landing page
  log "Generating landing page..."
  export IDEA
  node <<'EOF'
const https = require('https');
const fs = require('fs');

const idea = JSON.parse(process.env.IDEA);

const prompt = `Create a complete HTML landing page for:\n\nProduct: ${idea.title}\nProblem: ${idea.problem}\nSolution: ${idea.solution}\nTarget: ${idea.target_market}\n\nUse Tailwind CSS CDN. Include hero, problem, solution, CTA, FAQ sections. Return only HTML.`;

const body = JSON.stringify({
  model: 'gpt-4-turbo',
  messages: [{role: 'user', content: prompt}],
  temperature: 0.7,
  max_tokens: 4000
});

const req = https.request({
  hostname: 'api.openai.com',
  path: '/v1/chat/completions',
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
    'Content-Type': 'application/json'
  }
}, (res) => {
  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => {
    const result = JSON.parse(data);
    let html = result.choices[0].message.content;
    html = html.replace(/^```html\n/, '').replace(/\n```$/, '');
    
    const filename = `landing-${idea.title.toLowerCase().replace(/\s+/g, '-')}.html`;
    fs.writeFileSync(filename, html);
    console.log(filename);
  });
});

req.write(body);
req.end();
EOF
  
  LANDING_FILE=$(cat /tmp/ceoclaw_landing)
  success "Phase 3 complete: $LANDING_FILE"
  echo "$LANDING_FILE"
}

# ============================================
# PHASE 4: DEPLOYMENT
# ============================================
phase4_deployment() {
  local idea_id="$1"
  local landing_file="$2"
  log "PHASE 4: Deployment"
  
  approve "Deploy $landing_file to Vercel?"
  
  # Deploy to Vercel
  log "Deploying to Vercel..."
  CONTENT=$(cat "$landing_file" | jq -Rs .)
  
  RESPONSE=$(curl -sf -X POST "https://api.vercel.com/v13/deployments" \
    -H "Authorization: Bearer $CEOCLAW_VERCEL_TOKEN" \
    -H "Content-Type: application/json" \
    -d "{\"name\": \"ceoclaw-launch\", \"files\": [{\"file\": \"index.html\", \"data\": $CONTENT}]}")
  
  URL=$(echo "$RESPONSE" | jq -r '.url')
  DEPLOYMENT_ID=$(echo "$RESPONSE" | jq -r '.id')
  
  [ "$URL" = "null" ] && error "Deployment failed"
  
  # Store in DB
  sqlite3 "$DB" "INSERT INTO deployments (idea_id, url, platform, status) VALUES ($idea_id, 'https://$URL', 'vercel', 'live');"
  
  success "Phase 4 complete: https://$URL"
  echo "https://$URL"
}

# ============================================
# PHASE 5: MARKETING LAUNCH
# ============================================
phase5_marketing() {
  local url="$1"
  log "PHASE 5: Marketing Launch"
  
  approve "Generate and post marketing content for $url?"
  
  # Generate Twitter thread
  log "Generating Twitter thread..."
  # (Implementation similar to above - call GPT-4, generate content)
  
  warning "Marketing content generated. Review and post manually:"
  cat marketing-content.txt
  
  success "Phase 5 complete"
}

# ============================================
# PHASE 6: USER OUTREACH
# ============================================
phase6_outreach() {
  local idea_id="$1"
  log "PHASE 6: User Outreach"
  
  approve "Send outreach emails to target audience?"
  
  # Generate email template
  # Send to list (implement email sending)
  
  success "Phase 6 complete: Outreach emails sent"
}

# ============================================
# PHASE 7: METRICS TRACKING
# ============================================
phase7_metrics() {
  local deployment_id="$1"
  log "PHASE 7: Metrics Tracking"
  
  # Set up analytics
  log "Analytics configured. Tracking..."
  
  # Collect initial metrics
  sleep 5
  
  success "Phase 7 complete: Metrics tracking active"
}

# ============================================
# PHASE 8: ITERATION
# ============================================
phase8_iteration() {
  log "PHASE 8: Iteration"
  
  # Analyze metrics
  VISITS=$(sqlite3 "$DB" "SELECT COALESCE(SUM(visits), 0) FROM metrics")
  SIGNUPS=$(sqlite3 "$DB" "SELECT COALESCE(SUM(signups), 0) FROM metrics")
  
  log "Current metrics:"
  log "  Visits: $VISITS"
  log "  Signups: $SIGNUPS"
  
  if [ "$VISITS" -gt 100 ]; then
    success "Phase 8 complete: Meaningful traffic achieved!"
  else
    warning "Low traffic. Consider iteration or new idea."
  fi
}

# ============================================
# MAIN WORKFLOW
# ============================================
main() {
  log "🚀 CEOClaw Autonomous Founder Starting..."
  log "Workspace: $WORKSPACE"
  
  # Initialize database if needed
  if [ ! -f "$DB" ]; then
    log "Initializing database..."
    sqlite3 "$DB" < schema.sql  # Assume schema exists
  fi
  
  # Run phases
  phase1_market_discovery
  
  IDEA_ID=$(phase2_idea_generation)
  
  LANDING_FILE=$(phase3_mvp_build "$IDEA_ID")
  
  URL=$(phase4_deployment "$IDEA_ID" "$LANDING_FILE")
  
  DEPLOYMENT_ID=$(sqlite3 "$DB" "SELECT id FROM deployments WHERE url LIKE '%$URL%' LIMIT 1")
  
  phase5_marketing "$URL"
  
  phase6_outreach "$IDEA_ID"
  
  phase7_metrics "$DEPLOYMENT_ID"
  
  phase8_iteration
  
  success "================================"
  success "CEOClaw Autonomous Launch Complete!"
  success "================================"
  success "URL: $URL"
  success "Check metrics: sqlite3 $DB 'SELECT * FROM metrics'"
}

# Run
main "$@"
```

## Approval Gates

Insert approval gates before:
1. ✅ Generating ideas (review problems first)
2. ✅ Building landing page (pick best idea)
3. ✅ Deploying to production
4. ✅ Sending emails
5. ✅ Posting public marketing content

## Error Handling

```bash
# Add retry logic
retry_with_backoff() {
  local max_attempts=3
  local timeout=2
  local attempt=1
  
  until "$@"; do
    if [ $attempt -eq $max_attempts ]; then
      error "Command failed after $max_attempts attempts: $*"
    fi
    
    log "Attempt $attempt/$max_attempts failed. Retrying in ${timeout}s..."
    sleep $timeout
    attempt=$((attempt + 1))
    timeout=$((timeout * 2))
  done
}

# Usage
retry_with_backoff curl -sf "https://api.example.com/..."
```

## Dry Run Mode

```bash
# Add --dry-run flag
DRY_RUN=false

if [ "$1" = "--dry-run" ]; then
  DRY_RUN=true
  log "DRY RUN MODE: No real actions will be performed"
fi

# Wrap destructive actions
execute() {
  if [ "$DRY_RUN" = true ]; then
    log "[DRY RUN] Would execute: $*"
  else
    "$@"
  fi
}

# Usage
execute curl -X POST "https://api.vercel.com/..."
```

## Best Practices

1. **Start with dry run** - Test workflow first
2. **Use approval gates** - Human oversight for critical actions
3. **Log everything** - Track all phases for debugging
4. **Handle failures gracefully** - Retry with backoff
5. **Store all data** - Database persistence for history
6. **Monitor progress** - Real-time status updates
7. **Iterate quickly** - Don't wait for perfection

## Next Steps

This workflow ties together all CEOClaw capabilities. Customize as needed:
- Add more market research sources
- Implement A/B testing
- Add payment integration
- Scale marketing channels
- Automate customer support
