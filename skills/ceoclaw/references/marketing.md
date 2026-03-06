# Marketing Guide

Generate and distribute marketing content across channels to drive traffic to your launch.

## Overview

Marketing channels for early-stage products:
- **Content Marketing** - Blog posts, guides, case studies
- **Social Media** - Twitter/X, LinkedIn, Reddit, HackerNews
- **Email Marketing** - Newsletters, drip campaigns
- **Paid Ads** - Google Ads, Facebook Ads (optional)
- **Community** - ProductHunt, IndieHackers, relevant forums

## GPT-4 Content Generation

### Twitter Thread Template

```bash
# Generate viral Twitter thread
node <<'EOF'
const https = require('https');

const prompt = `Create a Twitter/X thread announcing a product launch.

PRODUCT:
- Name: ${process.env.PRODUCT_NAME}
- Problem: ${process.env.PROBLEM}
- Solution: ${process.env.SOLUTION}
- URL: ${process.env.URL}

Write a 7-tweet thread:
1. Hook (problem + intrigue)
2. Pain elaboration
3. Solution reveal
4. How it works
5. Key benefit
6. Call-to-action
7. Launch details

Rules:
- First tweet must hook in <10 words
- Use line breaks for readability
- Include emojis sparingly
- End with URL and hashtags
- Make it conversational, not salesy

Return as JSON array: [{tweet: "..."}]`;

// ... OpenAI API call ...
EOF
```

### Blog Post Generation

```bash
# Long-form content
node <<'EOF'
const prompt = `Write a 1000-word blog post.

TOPIC: ${process.env.TOPIC}
KEYWORDS: ${process.env.KEYWORDS}
AUDIENCE: ${process.env.AUDIENCE}

Structure:
1. Compelling headline
2. Introduction (hook + promise)
3. Problem explanation (3 sections)
4. Solution walkthrough
5. How it works (step-by-step)
6. Results/benefits
7. CTA with URL

Include:
- Subheadings (H2, H3)
- Short paragraphs (2-3 sentences)
- Bullet points
- Real examples
- Conversational tone

Return markdown format.`;
// ... API call ...
EOF
```

## Social Media Distribution

### Twitter/X Posting

```bash
# Post via Twitter API v2
TWEET_TEXT="Your tweet here"

curl -X POST "https://api.twitter.com/2/tweets" \
  -H "Authorization: Bearer $TWITTER_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"text\": \"$TWEET_TEXT\"}"
```

### LinkedIn Post

```bash
# LinkedIn API requires OAuth setup
# Simpler: Use Buffer or Hootsuite API

curl -X POST "https://api.bufferapp.com/1/updates/create.json" \
  -d "access_token=$BUFFER_TOKEN" \
  -d "profile_ids[]=$LINKEDIN_PROFILE_ID" \
  -d "text=$POST_TEXT" \
  -d "link=$URL"
```

### Reddit Submission

```bash
# Organic only (no API spam)
# Post manually to relevant subreddits:
# - /r/SideProject
# - /r/AlphaandBetausers  
# - /r/startups (Saturdays only)
# - Industry-specific subs

# Follow each subreddit's rules
# No spam - provide value
# Engage with comments
```

### HackerNews Show HN

```bash
# Manual submission required
# Title format: "Show HN: Product Name – One-line description"
# URL: Your landing page
# Timing: Tuesday-Thursday, 8-10am PT
```

## ProductHunt Launch

### Preparation Checklist

```markdown
## 2 Weeks Before
- [ ] Create ProductHunt account
- [ ] Build email list of supporters
- [ ] Prepare assets (logo, screenshots, video)
- [ ] Write tagline (<60 chars)
- [ ] Write description (clear, benefit-focused)

## 1 Week Before
- [ ] Schedule launch date (Tuesday-Thursday best)
- [ ] Line up "hunters" or self-submit
- [ ] Prepare comment responses
- [ ] Set up landing page tracking

## Launch Day
- [ ] Go live at 12:01am PT
- [ ] Reply to every comment quickly
- [ ] Share on social media
- [ ] Email supporters to upvote
- [ ] Monitor throughout day

## Post-Launch
- [ ] Thank supporters
- [ ] Share results
- [ ] Follow up with interested users
```

### ProductHunt API (for tracking, not submission)

```bash
# Get product stats
curl "https://api.producthunt.com/v2/api/graphql" \
  -H "Authorization: Bearer $PH_TOKEN" \
  -d '{"query": "{ post(id: \"product-id\") { votesCount commentsCount } }"}'
```

## Content Calendar Template

```bash
# Generate 30-day content plan
cat > content-calendar.md <<'EOF'
# 30-Day Launch Marketing Calendar

## Week 1: Pre-Launch Buzz
- Mon: Twitter teaser thread
- Wed: LinkedIn post (problem focus)
- Fri: Blog post on industry problem
- Sat: Reddit discussion thread

## Week 2: Launch Week
- Mon: Email waitlist (48h warning)
- Tue: ProductHunt launch
- Tue: Twitter launch thread
- Wed: LinkedIn launch post
- Thu: HackerNews Show HN
- Fri: Email waitlist (now live)

## Week 3: Growth
- Mon: Case study blog post
- Wed: Twitter tips thread
- Fri: YouTube demo video

## Week 4: Iteration
- Mon: Feature update post
- Wed: User testimonial thread
- Fri: Behind-the-scenes post
EOF
```

## Email Marketing Campaigns

See `email-outreach.md` for detailed implementation.

**Drip Campaign Structure:**
1. Welcome email (immediate)
2. Value/education (Day 2)
3. Feature highlight (Day 5)
4. Social proof (Day 7)
5. Conversion ask (Day 10)

## Paid Advertising (Optional)

### Google Ads Setup

```bash
# Requires Google Ads account setup
# Keywords targeting:
# - "[competitor] alternative"
# - "[problem] solution"
# - "[use case] tool"

# Budget: Start with $10-20/day
# Focus on high-intent keywords
# A/B test ad copy
```

### Facebook/Instagram Ads

```bash
# Good for B2C products
# Start with interest targeting
# Lookalike audiences from email list
# Budget: $10-15/day
# Focus on single conversion goal
```

## Analytics Tracking

Add UTM parameters to all shared links:

```bash
# Generate UTM links
generate_utm_link() {
  local base_url="$1"
  local source="$2"
  local medium="$3"
  local campaign="$4"
  
  echo "${base_url}?utm_source=${source}&utm_medium=${medium}&utm_campaign=${campaign}"
}

# Examples
generate_utm_link "https://yoursite.com" "twitter" "social" "launch"
generate_utm_link "https://yoursite.com" "producthunt" "referral" "launch"
generate_utm_link "https://yoursite.com" "email" "newsletter" "week1"
```

## Best Practices

1. **Be authentic** - No hype, just honest value
2. **Provide value first** - Don't just promote
3. **Engage with audience** - Reply to comments
4. **Track everything** - Use UTM parameters
5. **Test messaging** - A/B test headlines and copy
6. **Be consistent** - Post regularly
7. **Focus on one channel** - Master before expanding

## Content Examples

###Good Tweet Example:
```
I spent 5 hours/week researching competitors.

Now I spend 10 minutes.

Here's the tool I built to automate it:

[product name + 1-line value prop]

✅ Auto-scrapes 50+ sources
✅ Daily digests
✅ Sentiment analysis

Try free: [link]

#SaaS #productivity
```

### Bad Tweet Example:
```
Check out our revolutionary platform! 🚀
The future of productivity is here!
Sign up now! [link]
```

## Troubleshooting

**Low engagement on posts:**
- Post during peak hours (9-11am, 1-3pm local time)
- Use questions/hooks
- Add visual content
- Engage with others first

**Shadowbanned on Reddit:**
- Don't spam links
- Build karma organically
- Follow subreddit rules
- Add value, don't just promote

**No ProductHunt upvotes:**
- Launch Tuesday-Thursday (not Mon/Fri)
- Go live exactly at 12:01am PT
- Have supporters ready
- Respond to every comment
- Share broadly on launch day

## Next Steps

After marketing campaigns:
1. Track referral sources (see `metrics.md`)
2. Engage with interested users
3. Convert signups to paying customers
4. Iterate based on feedback
