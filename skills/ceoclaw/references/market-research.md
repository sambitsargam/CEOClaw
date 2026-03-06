# Market Research Guide

Scrape Reddit, HackerNews, and ProductHunt to discover user problems and pain points in your target market.

## Overview

Market research identifies problems worth solving by analyzing:
- Pain points discussed in online communities
- Feature requests and complaints
- Upvotes/engagement as demand signals
- Keyword patterns indicating common issues

## Data Sources

### 1. Reddit (JSON API)

Reddit provides a public JSON API (no auth required for basic access):

```bash
# Fetch top posts from a subreddit
curl -A "CEOClaw/1.0 (Market Research Bot)" \
  "https://www.reddit.com/r/SaaS/top.json?limit=100&t=month"

# Search specific keywords
curl -A "CEOClaw/1.0" \
  "https://www.reddit.com/r/startups/search.json?q=pain+point&limit=100&sort=top&t=month"

# Multiple subreddits
for sub in SaaS startups Entrepreneur indiebiz; do
  curl -A "CEOClaw/1.0" \
    "https://www.reddit.com/r/$sub/top.json?limit=100&t=month" \
    > "reddit_${sub}.json"
  sleep 2  # Rate limiting
done
```

**Rate Limits:** 60 requests/min (no auth), 100/min (with OAuth)

**Best Subreddits for B2B/SaaS:**
- /r/SaaS — software products discussion
- /r/startups — founder problems
- /r/Entrepreneur — business challenges
- /r/indiebiz — small business owners
- /r/smallbusiness — SMB pain points
- /r/webdev — developer tools
- /r/marketing — marketing challenges

### 2. HackerNews (Algolia API)

HN provides a free Algolia search API:

```bash
# Search "Ask HN" posts
curl "https://hn.algolia.com/api/v1/search?query=Ask%20HN&tags=ask_hn&numericFilters=created_at_i>$(date -v-30d +%s)"

# Search by keyword
curl "https://hn.algolia.com/api/v1/search?query=productivity+tool&tags=story"

# Get top stories with comments
curl "https://hn.algolia.com/api/v1/search?tags=front_page&hitsPerPage=50"
```

**Best Queries:**
- "Ask HN: What tools do you use for..."
- "Show HN: I built..."
- Tags: `ask_hn`, `show_hn`, `story`

**API Docs:** https://hn.algolia.com/api

### 3. ProductHunt (Web Scraping)

PH no longer has public API, use web scraping:

```bash
# Scrape homepage (requires HTML parsing)
curl -A "Mozilla/5.0" "https://www.producthunt.com"

# Or use Node.js with cheerio
node -e "
const https = require('https');
const cheerio = require('cheerio');

https.get('https://www.producthunt.com', (res) => {
  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => {
    const $ = cheerio.load(data);
    $('.item').each((i, el) => {
      const name = $(el).find('.name').text();
      const tagline = $(el).find('.tagline').text();
      const votes = $(el).find('.votes').text();
      console.log(JSON.stringify({name, tagline, votes}));
    });
  });
});
"
```

**Alternative:** Use ProductHunt's unofficial RSS feed:
```bash
curl "https://www.producthunt.com/feed"
```

## Data Extraction Workflow

### Step 1: Fetch Raw Data

```bash
#!/bin/bash
# market-research.sh

RESEARCH_DIR=~/.openclaw/ceoclaw/research
mkdir -p "$RESEARCH_DIR"
cd "$RESEARCH_DIR"

# Reddit
echo "Fetching Reddit data..."
curl -A "CEOClaw/1.0" \
  "https://www.reddit.com/r/SaaS/top.json?limit=100&t=month" \
  > reddit_saas.json
sleep 2

curl -A "CEOClaw/1.0" \
  "https://www.reddit.com/r/startups/search.json?q=frustrated+OR+annoying+OR+tedious&limit=100" \
  > reddit_startups_pain.json
sleep 2

# HackerNews
echo "Fetching HackerNews data..."
curl "https://hn.algolia.com/api/v1/search?query=Ask%20HN&tags=ask_hn&hitsPerPage=100" \
  > hn_ask.json

curl "https://hn.algolia.com/api/v1/search?query=productivity&tags=story&hitsPerPage=100" \
  > hn_productivity.json

echo "✅ Raw data fetched to $RESEARCH_DIR"
```

### Step 2: Parse and Extract Problems

Use `jq` or Node.js to parse JSON:

```bash
# Extract Reddit post titles and scores
jq -r '.data.children[] | 
  select(.data.score > 50) | 
  "\(.data.score)|\(.data.title)|\(.data.url)"' \
  reddit_saas.json > reddit_problems.txt

# Extract HN titles and points
jq -r '.hits[] | 
  select(.points > 20) | 
  "\(.points)|\(.title)|\(.url)"' \
  hn_ask.json > hn_problems.txt
```

**Or use GPT-4 to extract pain points:**

```bash
# Send to OpenAI API for intelligent extraction
node <<'EOF'
const fs = require('fs');
const https = require('https');

const data = fs.readFileSync('reddit_saas.json', 'utf8');
const posts = JSON.parse(data).data.children.map(c => ({
  title: c.data.title,
  selftext: c.data.selftext,
  score: c.data.score,
  url: c.data.url
}));

const prompt = `Extract user pain points from these Reddit posts. 
For each pain point, provide:
1. Clear problem description
2. User quote (if available)
3. Pain level (1-10 based on language intensity and upvotes)
4. Keywords

Posts:
${JSON.stringify(posts.slice(0, 20), null, 2)}

Return JSON array: [{problem, quote, pain_level, keywords}]`;

const req = https.request({
  hostname: 'api.openai.com',
  path: '/v1/chat/completions',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`
  }
}, (res) => {
  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => {
    const result = JSON.parse(data);
    const content = result.choices[0].message.content;
    fs.writeFileSync('extracted_problems.json', content);
    console.log('✅ Problems extracted to extracted_problems.json');
  });
});

req.write(JSON.stringify({
  model: 'gpt-4',
  messages: [{role: 'user', content: prompt}],
  temperature: 0.3
}));
req.end();
EOF
```

### Step 3: Score and Rank Problems

Scoring criteria:
- **Engagement** (upvotes/points): Higher = more people care
- **Recency**: Recent posts = active problem
- **Pain Language**: Words like "frustrated", "impossible", "nightmare" = high pain
- **Specificity**: Clear problem descriptions score higher

```bash
# Simple scoring with jq
jq -r '.[] | 
  .score = (.pain_level * 10 + .engagement * 0.1) | 
  . ' extracted_problems.json | \
jq -s 'sort_by(-.score) | .[:10]' > top_problems.json
```

### Step 4: Store in Database

```bash
# Insert into SQLite
cd ~/.openclaw/ceoclaw

while IFS='|' read -r score title url; do
  sqlite3 ceoclaw.db <<SQL
INSERT INTO market_signals (source, problem, pain_level, url, engagement, keywords)
VALUES (
  'reddit',
  '$title',
  $(( score / 10 )),
  '$url',
  $score,
  ''
);
SQL
done < research/reddit_problems.txt

echo "✅ Stored $(wc -l < research/reddit_problems.txt) signals in database"
```

## Search Patterns

### High-Intent Queries

**Problem Discovery:**
- "frustrated with [topic]"
- "why is [task] so hard"
- "looking for a tool that"
- "does anyone know a better way to"
- "I wish there was a"

**Solution Research:**
- "alternative to [product]"
- "[product] is too expensive"
- "[product] sucks"
- "migrating from [product]"

**Validation:**
- "would you pay for"
- "Show HN: I built"
- "looking for beta testers"

### Subreddit Selection Strategy

**B2B SaaS:**
- Technical: /r/webdev, /r/programming, /r/devops
- Business: /r/SaaS, /r/startups, /r/Entrepreneur
- Vertical: /r/marketing, /r/sales, /r/freelance

**Consumer Products:**
- /r/productivity, /r/ADHD, /r/LifeProTips
- /r/BuyItForLife, /r/minimalism
- Hobby-specific: /r/photography, /r/fitness, etc.

## Example: Complete Research Script

```bash
#!/bin/bash
# complete-market-research.sh

set -e

RESEARCH_DIR=~/.openclaw/ceoclaw/research
DB=~/.openclaw/ceoclaw/ceoclaw.db

mkdir -p "$RESEARCH_DIR"

echo "🔍 Starting market research..."

# 1. Fetch Reddit data
echo "📡 Fetching Reddit..."
for sub in SaaS startups webdev; do
  curl -sf -A "CEOClaw/1.0" \
    "https://www.reddit.com/r/$sub/top.json?limit=100&t=month" \
    > "$RESEARCH_DIR/reddit_${sub}.json"
  sleep 2
done

# 2. Fetch HackerNews
echo "📡 Fetching HackerNews..."
curl -sf "https://hn.algolia.com/api/v1/search?query=Ask%20HN&tags=ask_hn&hitsPerPage=100" \
  > "$RESEARCH_DIR/hn_ask.json"

# 3. Extract problems with GPT-4
echo "🤖 Extracting pain points with GPT-4..."
node <<'EOF'
const fs = require('fs');
const https = require('https');

async function extractProblems(file, source) {
  const data = JSON.parse(fs.readFileSync(file, 'utf8'));
  
  let posts;
  if (source === 'reddit') {
    posts = data.data.children.map(c => ({
      title: c.data.title,
      text: c.data.selftext,
      score: c.data.score,
      url: c.data.url
    })).filter(p => p.score > 20);
  } else {
    posts = data.hits.map(h => ({
      title: h.title,
      text: h.story_text || '',
      score: h.points,
      url: h.url
    })).filter(p => p.score > 20);
  }

  // GPT-4 extraction logic here
  // (Simplified for brevity)
  
  return posts.map(p => ({
    source,
    problem: p.title,
    pain_level: Math.min(10, Math.floor(p.score / 10)),
    url: p.url,
    engagement: p.score,
    keywords: ''
  }));
}

// Process all files
const results = [];
results.push(...await extractProblems('research/reddit_SaaS.json', 'reddit'));
results.push(...await extractProblems('research/hn_ask.json', 'hackernews'));

fs.writeFileSync('research/extracted_problems.json', JSON.stringify(results, null, 2));
console.log(`✅ Extracted ${results.length} problems`);
EOF

# 4. Store in database
echo "💾 Storing in database..."
jq -r '.[] | [.source, .problem, .pain_level, .url, .engagement, .keywords] | @tsv' \
  "$RESEARCH_DIR/extracted_problems.json" | \
while IFS=$'\t' read -r source problem pain url engage keywords; do
  sqlite3 "$DB" <<SQL
INSERT INTO market_signals (source, problem, pain_level, url, engagement, keywords)
VALUES ('$source', '$(echo "$problem" | sed "s/'/''/g")', $pain, '$url', $engage, '$keywords');
SQL
done

# 5. Show top results
echo ""
echo "🏆 Top 10 Problems by Engagement:"
sqlite3 "$DB" <<SQL
SELECT 
  source,
  substr(problem, 1, 60) || '...' as problem,
  engagement,
  pain_level
FROM market_signals
ORDER BY engagement DESC
LIMIT 10;
SQL

echo ""
echo "✅ Market research complete!"
echo "   Database: $DB"
echo "   Raw files: $RESEARCH_DIR"
```

## Best Practices

1. **Respect Rate Limits**: Sleep 1-2 seconds between requests
2. **Use User-Agent**: Reddit blocks default curl UA
3. **Authenticate When Possible**: Higher rate limits with OAuth
4. **Filter by Engagement**: Focus on posts with >20 upvotes
5. **Look for Patterns**: Same problem mentioned multiple times = strong signal
6. **Recent Data**: Prioritize posts from last 30 days
7. **Multiple Sources**: Cross-reference Reddit + HN + PH for validation

## Troubleshooting

**429 Too Many Requests (Reddit):**
```bash
# Add rate limiting
sleep 2  # Between requests

# Or use OAuth for higher limits
curl -H "Authorization: Bearer $REDDIT_ACCESS_TOKEN" \
  -A "CEOClaw/1.0" \
  "https://oauth.reddit.com/r/SaaS/top?limit=100&t=month"
```

**Empty Results:**
- Check date range (use recent time periods)
- Verify keywords are spelled correctly
- Try broader search terms
- Check subreddit exists and is public

**JSON Parse Errors:**
Check response with `-v` flag:
```bash
curl -v "https://www.reddit.com/r/SaaS/top.json?limit=10"
```

## Output Format

Store extracted problems in standard format:

```json
{
  "source": "reddit|hackernews|producthunt",
  "problem": "Clear description of user pain point",
  "pain_level": 7,
  "keywords": ["keyword1", "keyword2"],
  "url": "https://...",
  "engagement": 145,
  "created_at": "2026-03-07T10:30:00Z"
}
```

## Next Steps

After market research:
1. Review top problems in database
2. Look for patterns and clusters
3. Validate demand with keyword volume tools
4. Move to Idea Generation phase (see `idea-generation.md`)
