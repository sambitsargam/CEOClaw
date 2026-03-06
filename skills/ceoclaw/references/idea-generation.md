# Idea Generation Guide

Synthesize validated startup ideas from market research signals using GPT-4 prompting strategies.

## Overview

Transform raw pain points into actionable business ideas with:
- Problem-solution mapping
- Target market definition
- Unique value propositions
- Business model hypotheses
- Scoring and prioritization

## GPT-4 Prompting Strategy

### Core Prompt Template

```
You are an expert startup advisor analyzing market research to generate business ideas.

MARKET SIGNALS:
{list of problems from database}

Generate 5 startup ideas that solve these problems. For each idea:

1. **Title**: Short, memorable name (2-4 words)
2. **Problem**: Which specific pain point does this solve?
3. **Solution**: How does your product solve it? (2-3 sentences)
4. **Target Market**: Who are the primary users? Be specific.
5. **Value Proposition**: Why would someone pay for this?
6. **Business Model**: How will it make money?
7. **Competitive Advantage**: What makes this different/better?
8. **MVP Scope**: What's the minimum viable version?
9. **Score**: Rate 1-100 based on:
   - Problem severity (30 points)
   - Market size (30 points)
   - Solution feasibility (20 points)
   - Competitive moat (20 points)

Return valid JSON array only:
[{
  "title": "...",
  "problem": "...",
  "solution": "...",
  "target_market": "...",
  "value_proposition": "...",
  "business_model": "...",
  "competitive_advantage": "...",
  "mvp_scope": "...",
  "score": 85
}]

Focus on ideas that can be built and tested quickly (1-2 weeks for MVP).
Prefer simple solutions over complex platforms.
```

### Implementation

```bash
# Get top problems from database
cd ~/.openclaw/ceoclaw

PROBLEMS=$(sqlite3 ceoclaw.db <<SQL
SELECT 
  json_group_array(
    json_object(
      'problem', problem,
      'source', source,
      'engagement', engagement,
      'pain_level', pain_level
    )
  )
FROM (
  SELECT * FROM market_signals 
  ORDER BY engagement DESC 
  LIMIT 20
);
SQL
)

# Generate ideas with GPT-4
node <<'EOF'
const https = require('https');
const fs = require('fs');

const problems = process.env.PROBLEMS;

const prompt = `You are an expert startup advisor analyzing market research to generate business ideas.

MARKET SIGNALS:
${problems}

Generate 5 startup ideas that solve these problems. For each idea:

1. **Title**: Short, memorable name (2-4 words)
2. **Problem**: Which specific pain point does this solve?
3. **Solution**: How does your product solve it? (2-3 sentences)
4. **Target Market**: Who are the primary users? Be specific.
5. **Value Proposition**: Why would someone pay for this?
6. **Business Model**: How will it make money?
7. **Competitive Advantage**: What makes this different/better?
8. **MVP Scope**: What's the minimum viable version?
9. **Score**: Rate 1-100 based on problem severity (30), market size (30), feasibility (20), moat (20)

Return ONLY valid JSON array:
[{
  "title": "...",
  "problem": "...",
  "solution": "...",
  "target_market": "...",
  "value_proposition": "...",
  "business_model": "...",
  "competitive_advantage": "...",
  "mvp_scope": "...",
  "score": 85
}]

Focus on ideas that can be built and tested in 1-2 weeks.`;

const requestBody = JSON.stringify({
  model: 'gpt-4-turbo',
  messages: [{ role: 'user', content: prompt }],
  temperature: 0.7,
  response_format: { type: 'json_object' }
});

const req = https.request({
  hostname: 'api.openai.com',
  path: '/v1/chat/completions',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
    'Content-Length': Buffer.byteLength(requestBody)
  }
}, (res) => {
  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => {
    const result = JSON.parse(data);
    if (result.error) {
      console.error('OpenAI Error:', result.error);
      process.exit(1);
    }
    
    const content = result.choices[0].message.content;
    const ideas = JSON.parse(content);
    
    // Write to file
    fs.writeFileSync('ideas.json', JSON.stringify(ideas, null, 2));
    
    // Insert into database
    const sqlite3 = require('child_process').spawnSync('sqlite3', ['ceoclaw.db'], {
      input: ideas.map(idea => `
        INSERT INTO ideas (title, description, target_market, solution, score, status)
        VALUES (
          '${idea.title.replace(/'/g, "''")}',
          '${idea.problem.replace(/'/g, "''")}',
          '${idea.target_market.replace(/'/g, "''")}',
          '${idea.solution.replace(/'/g, "''")}',
          ${idea.score},
          'generated'
        );
      `).join('\n')
    });
    
    console.log(`✅ Generated ${ideas.length} ideas`);
    console.log('\nTop 3 Ideas:');
    ideas.sort((a, b) => b.score - a.score).slice(0, 3).forEach((idea, i) => {
      console.log(`\n${i + 1}. ${idea.title} (Score: ${idea.score})`);
      console.log(`   ${idea.solution}`);
    });
  });
});

req.on('error', (e) => {
  console.error('Request error:', e);
  process.exit(1);
});

req.write(requestBody);
req.end();
EOF
```

## Scoring Framework

### Problem Severity (30 points)
- 25-30: Critical blocker (business can't function)
- 20-24: Major pain (costs time/money daily)
- 15-19: Moderate annoyance (weekly impact)
- 10-14: Minor inconvenience
- 0-9: Nice-to-have

### Market Size (30 points)
- 25-30: Huge market (millions of potential users)
- 20-24: Large market (hundreds of thousands)
- 15-19: Medium market (tens of thousands)
- 10-14: Niche market (thousands)
- 0-9: Very niche (hundreds)

### Solution Feasibility (20 points)
- 17-20: Simple to build (1-2 weeks)
- 13-16: Moderate complexity (1 month)
- 9-12: Complex (2-3 months)
- 5-8: Very complex (6+ months)
- 0-4: Requires major infrastructure

### Competitive Moat (20 points)
- 17-20: Strong defensibility (network effects, data moat)
- 13-16: Moderate differentiation
- 9-12: Copyable but execution matters
- 5-8: Easy to copy
- 0-4: Commodity

## Validation Checklist

Before committing to an idea, validate:

### 1. Demand Validation
```bash
# Search keyword volume (use Google Trends API or manual)
# Check competition on Google
curl -A "Mozilla/5.0" "https://www.google.com/search?q=your+product+category" | \
  grep -o "About [0-9,]* results" | \
  grep -o "[0-9,]*"

# Check Reddit mentions
curl -A "CEOClaw/1.0" \
  "https://www.reddit.com/search.json?q=your+product+category&limit=100"
```

### 2. Competition Analysis
- Google the target keywords
- Check ProductHunt for similar products
- Search "alternative to [competitor]"
- Check Y Combinator companies

### 3. Willingness to Pay
Look for evidence people pay for solutions:
- Existing paid products in the space
- Reddit/HN posts asking for recommendations
- Freemium model with paid tiers
- B2B SaaS indicators

### 4. Technical Feasibility
Can you build an MVP in 1-2 weeks?
- No AI training required
- Existing APIs available
- Simple tech stack
- No complex infrastructure

## Idea Refinement Prompts

### Prompt 1: Competitive Analysis
```
Analyze competitors for this idea:

IDEA: {your idea}

Research:
1. List 5 direct competitors
2. Their pricing models
3. What they do well
4. Their weaknesses/gaps
5. How we can differentiate

Return structured JSON.
```

### Prompt 2: MVP Scoping
```
Define minimal MVP for this idea:

IDEA: {your idea}

Create a 1-week MVP scope:
1. Core features (must-have only)
2. What to skip for V1
3. Tech stack recommendation
4. Development timeline
5. Success metrics

Keep it ruthlessly simple. What's the smallest version that proves value?
```

### Prompt 3: Landing Page Copy
```
Write landing page copy for:

IDEA: {your idea}
TARGET: {target market}

Include:
1. Hero headline (10 words max)
2. Subheadline (1 sentence value prop)
3. 3 key benefits
4. Call-to-action
5. Social proof idea

Make it clear, specific, and benefit-focused.
```

## Example: Full Idea Generation Flow

```bash
#!/bin/bash
# generate-ideas.sh

set -e

DB=~/.openclaw/ceoclaw/ceoclaw.db
OPENAI_KEY="${OPENAI_API_KEY:?OPENAI_API_KEY not set}"

echo "💡 Generating startup ideas from market research..."

# 1. Get top problems
echo "📊 Fetching market signals..."
PROBLEMS=$(sqlite3 "$DB" <<SQL
SELECT json_group_array(
  json_object(
    'problem', problem,
    'source', source,
    'engagement', engagement,
    'pain_level', pain_level,
    'url', url
  )
)
FROM (
  SELECT * FROM market_signals 
  ORDER BY (engagement * pain_level) DESC 
  LIMIT 20
);
SQL
)

# 2. Call GPT-4
echo "🤖 Calling GPT-4 for idea generation..."
RESPONSE=$(curl -sf https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_KEY" \
  -d @- <<JSON
{
  "model": "gpt-4-turbo",
  "messages": [{
    "role": "user",
    "content": "You are a startup advisor. Generate 5 business ideas from these problems:\n\n$PROBLEMS\n\nFor each: title, problem, solution, target_market, value_proposition, business_model, competitive_advantage, mvp_scope, score (1-100). Return JSON array only."
  }],
  "temperature": 0.7
}
JSON
)

# 3. Parse response
IDEAS=$(echo "$RESPONSE" | jq -r '.choices[0].message.content')

# 4. Store in database
echo "💾 Storing ideas..."
echo "$IDEAS" | jq -c '.[]' | while read -r idea; do
  TITLE=$(echo "$idea" | jq -r '.title')
  PROBLEM=$(echo "$idea" | jq -r '.problem' | sed "s/'/''/g")
  SOLUTION=$(echo "$idea" | jq -r '.solution' | sed "s/'/''/g")
  TARGET=$(echo "$idea" | jq -r '.target_market' | sed "s/'/''/g")
  SCORE=$(echo "$idea" | jq -r '.score')
  
  sqlite3 "$DB" <<SQL
INSERT INTO ideas (title, description, target_market, solution, score, status)
VALUES ('$TITLE', '$PROBLEM', '$TARGET', '$SOLUTION', $SCORE, 'generated');
SQL
done

# 5. Display results
echo ""
echo "✅ Generated ideas:"
sqlite3 "$DB" <<SQL
.mode column
.headers on
SELECT 
  id,
  title,
  score,
  substr(solution, 1, 50) || '...' as solution
FROM ideas
WHERE status = 'generated'
ORDER BY score DESC;
SQL

echo ""
echo "🎯 Next: Review ideas and pick one to build"
echo "   Command: sqlite3 $DB 'SELECT * FROM ideas WHERE id=X'"
```

## Idea Selection Criteria

When choosing which idea to pursue:

### High-Priority Signals
✅ Score >70
✅ Clear single-user pain point
✅ Can build MVP in <2 weeks
✅ Monetization path obvious
✅ 10+ market research mentions
✅ Existing competition (validates market)
✅ Simple tech stack

### Red Flags
❌ Score <50
❌ Requires regulatory approval
❌ Multi-sided marketplace (chicken/egg)
❌ Requires 100k+ users to work
❌ Complex AI/ML training
❌ No existing competition (unvalidated)
❌ Vague value proposition

## Advanced: Multi-Round Refinement

For top ideas, run additional refinement:

```bash
# Get top idea
TOP_IDEA=$(sqlite3 "$DB" "SELECT json_object('title', title, 'solution', solution, 'target_market', target_market) FROM ideas ORDER BY score DESC LIMIT 1")

# Refine with focused prompts
for stage in "competitive_analysis" "mvp_scope" "landing_copy" "pricing_strategy"; do
  echo "Refining: $stage..."
  
  RESPONSE=$(curl -sf https://api.openai.com/v1/chat/completions \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -H "Content-Type: application/json" \
    -d @- <<JSON
{
  "model": "gpt-4-turbo",
  "messages": [{
    "role": "user",
    "content": "For this startup idea:\n\n$TOP_IDEA\n\nProvide detailed $stage analysis. Be specific and actionable."
  }],
  "temperature": 0.5
}
JSON
  )
  
  echo "$RESPONSE" | jq -r '.choices[0].message.content' > "refinement_${stage}.txt"
  sleep 1
done

echo "✅ Refinement complete. Check refinement_*.txt files"
```

## Best Practices

1. **Generate Multiple Ideas**: Don't settle on first idea
2. **Cross-Reference Problems**: Look for patterns across sources
3. **Score Objectively**: Use scoring framework consistently
4. **Validate Assumptions**: Check market size and willingness to pay
5. **Keep MVP Scope Small**: Prioritize speed to market
6. **Focus on Hair-on-Fire Problems**: Urgent > nice-to-have
7. **Document Reasoning**: Store why you picked/rejected ideas

## Next Steps

After idea generation and selection:
1. Create detailed MVP specification (see `landing-pages.md`)
2. Build landing page to test messaging
3. Set up deployment pipeline (see `deployment.md`)
4. Prepare marketing materials (see `marketing.md`)

## Troubleshooting

**GPT-4 returns invalid JSON:**
- Add `response_format: { type: 'json_object' }`
- Use explicit "Return ONLY valid JSON" in prompt
- Parse with error handling and retry

**Ideas are too generic:**
- Provide more specific market signals
- Include user quotes and context
- Request concrete examples in prompt
- Lower temperature (0.3-0.5)

**Low scores across all ideas:**
- Revisit market research
- Try different problem spaces
- Broaden target market
- Simplify solution scope
