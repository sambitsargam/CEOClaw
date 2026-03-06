# Metrics Tracking Guide

Monitor website analytics, user behavior, and business metrics to guide iteration.

## Overview

Track key metrics:
- **Traffic** - Visits, page views, referral sources
- **Engagement** - Time on site, bounce rate, pages per session
- **Conversion** - Signups, trials, purchases
- **Revenue** - MRR, churn, LTV

## Analytics Platform Setup

### Google Analytics 4 (Free)

1. Create property: https://analytics.google.com
2. Get measurement ID (G-XXXXXXXXXX)
3. Add to landing page:

```html
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

### Plausible Analytics (Privacy-focused)

```html
<!-- Plausible -->
<script defer data-domain="yourdomain.com" src="https://plausible.io/js/script.js"></script>
```

### Simple DIY Tracking

```html
<!-- Lightweight pixel tracking -->
<script>
  // Log pageview
  fetch('/api/track', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({
      event: 'pageview',
      page: window.location.pathname,
      referrer: document.referrer,
      timestamp: Date.now()
    })
  });
  
  // Track button clicks
  document.querySelectorAll('[data-track]').forEach(el => {
    el.addEventListener('click', e => {
      fetch('/api/track', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({
          event: 'click',
          element: e.target.dataset.track,
          timestamp: Date.now()
        })
      });
    });
  });
</script>
```

## Data Collection via API

### Google Analytics Reporting API

```bash
# Install Google Auth Library
npm install -g @google-cloud/analytics-data

# Or use REST API
ACCESS_TOKEN="your_oauth_token"
PROPERTY_ID="properties/123456789"

curl "https://analyticsdata.googleapis.com/v1beta/$PROPERTY_ID:runReport" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "dateRanges": [{"startDate": "7daysAgo", "endDate": "today"}],
    "metrics": [
      {"name": "activeUsers"},
      {"name": "sessions"},
      {"name": "screenPageViews"}
    ],
    "dimensions": [{"name": "date"}]
  }'
```

### Plausible API

```bash
# Get Plausible API key: https://plausible.io/settings
PLAUSIBLE_KEY="your_api_key"
SITE_ID="yourdomain.com"

# Get aggregate stats
curl "https://plausible.io/api/v1/stats/aggregate?site_id=$SITE_ID&period=7d&metrics=visitors,visits,pageviews,bounce_rate" \
  -H "Authorization: Bearer $PLAUSIBLE_KEY"

# Get timeseries
curl "https://plausible.io/api/v1/stats/timeseries?site_id=$SITE_ID&period=30d" \
  -H "Authorization: Bearer $PLAUSIBLE_KEY"
```

## Store Metrics in Database

```bash
#!/bin/bash
# fetch-metrics.sh

set -e

DB=~/.openclaw/ceoclaw/ceoclaw.db
DEPLOYMENT_ID="${1:?Usage: $0 <deployment-id>}"

# Fetch from analytics (example with Plausible)
STATS=$(curl -sf "https://plausible.io/api/v1/stats/aggregate?site_id=$SITE_ID&period=day&metrics=visitors,visits,pageviews,bounce_rate" \
  -H "Authorization: Bearer $PLAUSIBLE_KEY")

VISITS=$(echo "$STATS" | jq -r '.results.visits.value')
VISITORS=$(echo "$STATS" | jq -r '.results.visitors.value')
PAGEVIEWS=$(echo "$STATS" | jq -r '.results.pageviews.value')
BOUNCE=$(echo "$STATS" | jq -r '.results.bounce_rate.value')

# Store in database
sqlite3 "$DB" <<SQL
INSERT INTO metrics (deployment_id, date, visits, visitors, pageviews, bounce_rate, created_at)
VALUES (
  $DEPLOYMENT_ID,
  date('now'),
  $VISITS,
  $VISITORS,
  $PAGEVIEWS,
  $BOUNCE,
  datetime('now')
);
SQL

echo "✅ Metrics stored for deployment $DEPLOYMENT_ID"
echo "   Visits: $VISITS"
echo "   Visitors: $VISITORS"
echo "   Pageviews: $PAGEVIEWS"
echo "   Bounce: $BOUNCE%"
```

## Custom Event Tracking

### Track Signups

```html
<!-- In landing page form -->
<form onsubmit="trackSignup(event)">
  <input type="email" name="email" required>
  <button type="submit">Sign Up</button>
</form>

<script>
function trackSignup(e) {
  e.preventDefault();
  const email = e.target.email.value;
  
  // Send to your backend
  fetch('/api/signup', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({email})
  })
  .then(res => res.json())
  .then(data => {
    // Track conversion event
    if (window.gtag) {
      gtag('event', 'signup', {
        'event_category': 'conversion',
        'event_label': 'email_signup'
      });
    }
    
    // Or Plausible
    if (window.plausible) {
      plausible('Signup');
    }
    
    alert('Thanks for signing up!');
  });
}
</script>
```

### Track Button Clicks

```html
<button 
  onclick="trackClick('cta_hero')"
  data-track="cta_hero"
>
  Get Started
</button>

<script>
function trackClick(label) {
  if (window.gtag) {
    gtag('event', 'click', {
      'event_category': 'engagement',
      'event_label': label
    });
  }
}
</script>
```

## Automated Daily Reports

```bash
#!/bin/bash
# daily-metrics-report.sh

set -e

DB=~/.openclaw/ceoclaw/ceoclaw.db
TODAY=$(date +%Y-%m-%d)

# Fetch today's metrics (implement your analytics API call)
# ... (as shown above)

# Generate report
REPORT=$(sqlite3 "$DB" <<SQL
.mode column
.headers on
SELECT 
  d.url,
  m.visits,
  m.signups,
  m.conversions,
  ROUND(m.conversions * 100.0 / m.visits, 2) as conversion_rate
FROM deployments d
LEFT JOIN metrics m ON m.deployment_id = d.id
WHERE m.date = '$TODAY'
ORDER BY m.visits DESC;
SQL
)

echo "📊 Daily Metrics Report - $TODAY"
echo ""
echo "$REPORT"

# Email report (optional)
if [ -n "$CEOCLAW_REPORT_EMAIL" ]; then
  echo "$REPORT" | mail -s "CEOClaw Daily Report - $TODAY" "$CEOCLAW_REPORT_EMAIL"
fi
```

## Key Metrics Formulas

### Conversion Rate
```sql
SELECT 
  (signups * 100.0 / visits) as conversion_rate
FROM metrics
WHERE deployment_id = ?;
```

### Growth Rate (Week over Week)
```sql
SELECT 
  ((this_week.visits - last_week.visits) * 100.0 / last_week.visits) as growth_rate
FROM 
  (SELECT SUM(visits) as visits FROM metrics WHERE date >= date('now', '-7 days')) this_week,
  (SELECT SUM(visits) as visits FROM metrics WHERE date >= date('now', '-14 days') AND date < date('now', '-7 days')) last_week;
```

### Average Session Duration
Requires custom tracking with timestamps:
```javascript
// Track session start
localStorage.setItem('sessionStart', Date.now());

// Track session end (on page unload)
window.addEventListener('beforeunload', () => {
  const duration = Date.now() - localStorage.getItem('sessionStart');
  fetch('/api/track', {
    method: 'POST',
    body: JSON.stringify({event: 'session_duration', duration}),
    keepalive: true
  });
});
```

### Retention Rate
```sql
-- Users who returned within 7 days
SELECT 
  COUNT(DISTINCT returning.user_id) * 100.0 / COUNT(DISTINCT first.user_id) as retention_rate
FROM 
  (SELECT user_id, MIN(created_at) as first_visit FROM events GROUP BY user_id) first
LEFT JOIN 
  (SELECT user_id, created_at FROM events) returning
  ON first.user_id = returning.user_id 
  AND returning.created_at > first.first_visit 
  AND returning.created_at <= date(first.first_visit, '+7 days');
```

## Dashboard Queries

### Traffic Summary
```bash
sqlite3 ~/.openclaw/ceoclaw/ceoclaw.db <<SQL
.mode column
.headers on

SELECT 
  date,
  SUM(visits) as total_visits,
  SUM(signups) as total_signups,
  ROUND(AVG(bounce_rate), 2) as avg_bounce,
  ROUND(SUM(signups) * 100.0 / SUM(visits), 2) as conversion_rate
FROM metrics
WHERE date >= date('now', '-30 days')
GROUP BY date
ORDER BY date DESC;
SQL
```

### Top Referral Sources (requires UTM tracking)
```sql
SELECT 
  utm_source,
  COUNT(*) as visits,
  SUM(CASE WHEN converted = 1 THEN 1 ELSE 0 END) as conversions
FROM events
WHERE utm_source IS NOT NULL
GROUP BY utm_source
ORDER BY visits DESC
LIMIT 10;
```

### Funnel Analysis
```sql
-- Conversion funnel: Visit → Signup → Trial → Paid
SELECT 
  'Visits' as stage,
  COUNT(DISTINCT session_id) as count,
  100.0 as percentage
FROM events
WHERE event = 'pageview'

UNION ALL

SELECT 
  'Signups',
  COUNT(DISTINCT session_id),
  COUNT(DISTINCT session_id) * 100.0 / (SELECT COUNT(DISTINCT session_id) FROM events WHERE event = 'pageview')
FROM events
WHERE event = 'signup'

UNION ALL

SELECT 
  'Trials',
  COUNT(DISTINCT user_id),
  COUNT(DISTINCT user_id) * 100.0 / (SELECT COUNT(DISTINCT session_id) FROM events WHERE event = 'pageview')
FROM subscriptions
WHERE status = 'trial'

UNION ALL

SELECT 
  'Paid',
  COUNT(DISTINCT user_id),
  COUNT(DISTINCT user_id) * 100.0 / (SELECT COUNT(DISTINCT session_id) FROM events WHERE event = 'pageview')
FROM subscriptions
WHERE status = 'active';
```

## Visualization

### Generate Simple Charts with Gnuplot

```bash
#!/bin/bash
# visualize-metrics.sh

DB=~/.openclaw/ceoclaw/ceoclaw.db

# Export data for plotting
sqlite3 "$DB" <<SQL > metrics.dat
SELECT date, visits FROM metrics 
WHERE date >= date('now', '-30 days')
ORDER BY date;
SQL

# Generate chart
gnuplot <<EOF
set terminal png size 800,400
set output 'traffic-chart.png'
set title 'Traffic - Last 30 Days'
set xlabel 'Date'
set ylabel 'Visits'
set xdata time
set timefmt "%Y-%m-%d"
set format x "%m/%d"
set grid
plot 'metrics.dat' using 1:2 with lines lw 2 title 'Visits'
EOF

echo "📈 Chart generated: traffic-chart.png"
```

## Alerts & Notifications

```bash
#!/bin/bash
# metrics-alerts.sh

DB=~/.openclaw/ceoclaw/ceoclaw.db

# Check for anomalies
TODAY_VISITS=$(sqlite3 "$DB" "SELECT visits FROM metrics WHERE date = date('now') LIMIT 1")
AVG_VISITS=$(sqlite3 "$DB" "SELECT AVG(visits) FROM metrics WHERE date >= date('now', '-7 days')")

# Alert if traffic drops >50%
if (( $(echo "$TODAY_VISITS < $AVG_VISITS * 0.5" | bc) )); then
  echo "🚨 ALERT: Traffic dropped significantly!"
  echo "   Today: $TODAY_VISITS visits"
  echo "   7-day avg: $AVG_VISITS visits"
  
  # Send alert (email, Slack, etc.)
fi

# Alert on high conversion day
CONVERSION_RATE=$(sqlite3 "$DB" "SELECT ROUND(signups * 100.0 / visits, 2) FROM metrics WHERE date = date('now')")
if (( $(echo "$CONVERSION_RATE > 10" | bc) )); then
  echo "🎉 Great conversion rate today: $CONVERSION_RATE%!"
fi
```

## Privacy & GDPR Compliance

1. **Cookie Consent**: Add banner if using cookies
2. **Anonymize IPs**: Enable in GA4 settings
3. **Data Retention**: Set appropriate retention periods
4. **Privacy Policy**: Include analytics disclosure
5. **Opt-Out**: Provide mechanism for users

```html
<!-- Simple privacy-friendly tracking -->
<script defer data-no-cookie data-domain="yourdomain.com" src="https://plausible.io/js/script.js"></script>
```

## Best Practices

1. **Track the right metrics**:
   - Early stage: Traffic, signups, user feedback
   - Growth stage: Activation, retention, revenue
   - Don't track vanity metrics

2. **Set up goals/conversions**:
   - Signup completions
   - Trial starts
   - Purchase completions

3. **Monitor daily**:
   - Automate daily reports
   - Set up alerts for anomalies
   - Review weekly trends

4. **A/B test changes**:
   - Different headlines
   - CTA button colors/copy
   - Pricing tiers

## Troubleshooting

**Analytics not tracking:**
- Check script loads properly (view source)
- Verify measurement ID is correct
- Allow 24-48h for data to appear
- Check browser ad blockers aren't blocking

**Inflated traffic:**
- Filter out bot traffic
- Exclude internal IPs
- Check for referral spam

**Low data accuracy:**
- Compare multiple sources (GA + Plausible)
- Cross-reference with server logs
- Check timezone settings

## Next Steps

After collecting metrics:
1. Analyze which channels drive quality traffic
2. Identify drop-off points in funnel
3. Iterate on low-performing areas
4. Double down on what's working
5. Set goals and track progress
