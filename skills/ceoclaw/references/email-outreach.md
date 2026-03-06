# Email Outreach Guide

Send personalized email campaigns to early adopters and potential customers using SMTP.

## Overview

Email outreach strategies:
- **Cold Email** - Reaching out to prospects
- **Warm Email** - Following up with waitlist signups
- **Newsletter** - Regular updates to subscribers
- **Drip Campaigns** - Automated sequences

## SMTP Setup

### Gmail App Password

1. Enable 2FA: https://myaccount.google.com/security
2. Create app password: https://myaccount.google.com/apppasswords
3. Save credentials:

```bash
export CEOCLAW_SMTP_HOST="smtp.gmail.com"
export CEOCLAW_SMTP_PORT="587"
export CEOCLAW_SMTP_USER="your@gmail.com"
export CEOCLAW_SMTP_PASS="app-password-here"
```

### SendGrid API (Alternative)

```bash
# Get API key: https://app.sendgrid.com/settings/api_keys
export CEOCLAW_SENDGRID_KEY="SG.xxx"

# Send via API
curl -X POST "https://api.sendgrid.com/v3/mail/send" \
  -H "Authorization: Bearer $CEOCLAW_SENDGRID_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "personalizations": [{
      "to": [{"email": "user@example.com"}],
      "subject": "Your subject"
    }],
    "from": {"email": "you@yourdomain.com"},
    "content": [{
      "type": "text/html",
      "value": "<p>Your HTML content</p>"
    }]
  }'
```

## Email Template Generation

### GPT-4 Email Prompt

```bash
node <<'EOF'
const prompt = `Write a cold email for user outreach.

PRODUCT: ${process.env.PRODUCT_NAME}
TARGET: ${process.env.TARGET_PERSONA}
GOAL: ${process.env.EMAIL_GOAL}

Write a short, personalized email (100-150 words):

Subject line:
- Personalized to recipient
- Clear benefit (<50 chars)
- No spam triggers (FREE, !!!, URGENT)

Body:
1. Personal opener (show you researched them)
2. One-sentence problem acknowledgment
3. One-sentence solution (your product)
4. Specific value for them
5. Soft CTA (question, not "Buy now")
6. Sign off

Rules:
- Conversational, not corporate
- No hype or superlatives
- Specific benefits, not features
- Make it skimmable (short paragraphs)
- Include unsubscribe link

Return JSON: {subject: "...", body: "..."}`;

// ... OpenAI API call ...
EOF
```

## Sending Emails with Node.js

### Simple SMTP Sender

```bash
cd ~/.openclaw/ceoclaw

node <<'EOF'
const nodemailer = require('nodemailer');

// Create transporter
const transporter = nodemailer.createTransporter({
  host: process.env.CEOCLAW_SMTP_HOST,
  port: parseInt(process.env.CEOCLAW_SMTP_PORT || '587'),
  secure: false, // true for 465, false for other ports
  auth: {
    user: process.env.CEOCLAW_SMTP_USER,
    pass: process.env.CEOCLAW_SMTP_PASS
  }
});

// Send email
async function sendEmail(to, subject, html) {
  try {
    const info = await transporter.sendMail({
      from: `"Your Name" <${process.env.CEOCLAW_SMTP_USER}>`,
      to,
      subject,
      html
    });
    
    console.log(`✅ Sent to ${to}: ${info.messageId}`);
    return info;
  } catch (error) {
    console.error(`❌ Failed to send to ${to}:`, error.message);
    throw error;
  }
}

// Example usage
sendEmail(
  'user@example.com',
  'Quick question about your workflow',
  '<p>Hi there,</p><p>I noticed you...</p>'
);
EOF
```

### Bulk Email Script with Rate Limiting

```bash
#!/bin/bash
# bulk-email.sh

set -e

DB=~/.openclaw/ceoclaw/ceoclaw.db
CAMPAIGN_NAME="${1:?Usage: $0 <campaign-name>}"

# Get email list (implement your own source)
EMAILS=$(cat email-list.txt)  # One email per line

# Send with rate limiting
SENT=0
FAILED=0

for email in $EMAILS; do
  echo "Sending to: $email"
  
  # Call Node.js sender
  if node send-email.js "$email" "$CAMPAIGN_NAME"; then
    ((SENT++))
    
    # Log success
    sqlite3 "$DB" <<SQL
INSERT INTO outreach_log (email, campaign, status, sent_at)
VALUES ('$email', '$CAMPAIGN_NAME', 'sent', datetime('now'));
SQL
  else
    ((FAILED++))
    
    # Log failure
    sqlite3 "$DB" <<SQL
INSERT INTO outreach_log (email, campaign, status, sent_at)
VALUES ('$email', '$CAMPAIGN_NAME', 'failed', datetime('now'));
SQL
  fi
  
  # Rate limiting: 1 email per 2 seconds (30/min)
  sleep 2
done

echo ""
echo "✅ Campaign complete!"
echo "   Sent: $SENT"
echo "   Failed: $FAILED"
```

## Personalization Variables

Use merge tags for personalization:

```javascript
function personalize(template, data) {
  return template
    .replace(/\{\{name\}\}/g, data.name)
    .replace(/\{\{company\}\}/g, data.company)
    .replace(/\{\{problem\}\}/g, data.problem)
    .replace(/\{\{benefit\}\}/g, data.benefit);
}

// Example
const template = `Hi {{name}},

I saw that {{company}} is working on {{problem}}.

Would a tool that {{benefit}} be helpful?

Best,
Your Name`;

const email = personalize(template, {
  name: 'Alex',
  company: 'Acme Inc',
  problem: 'scaling customer support',
  benefit: 'automates 80% of support tickets'
});
```

## Email Templates

### Cold Outreach

```html
Subject: Quick question about {{company}}'s {{problem}}

Hi {{name}},

I noticed {{company}} is {{specific_observation}}.

We just launched {{product}} – it {{specific_benefit}}.

Would it be helpful if I sent you a quick demo?

Best,
{{your_name}}

---
Not interested? <a href="{{unsubscribe_link}}">Unsubscribe</a>
```

### Welcome Email (Waitlist Signups)

```html
Subject: Welcome to {{product}}! 🎉

Hi {{name}},

Thanks for joining {{product}}!

You're on the waitlist. We'll email you as soon as we launch (expected: {{launch_date}}).

In the meantime, here's what we're building:
- {{benefit_1}}
- {{benefit_2}}
- {{benefit_3}}

Have feedback? Just reply to this email.

Best,
{{founder_name}}
```

### Feature Update

```html
Subject: {{product}} update: {{new_feature}}

Hi {{name}},

Quick update: We just shipped {{new_feature}}.

What it does: {{feature_description}}

Why it matters: {{benefit}}

Try it now: {{cta_link}}

Let me know what you think!

Best,
{{your_name}}
```

## CAN-SPAM Compliance

**Required elements:**
1. ✅ Accurate "From" name and email
2. ✅ Truthful subject line
3. ✅ Physical mailing address in footer
4. ✅ Clear unsubscribe mechanism
5. ✅ Honor opt-outs within 10 business days

```html
<!-- Footer template -->
<div style="margin-top: 40px; padding-top: 20px; border-top: 1px solid #ccc; color: #666; font-size: 12px;">
  <p>{{Company Name}}<br>{{Address}}<br>{{City, State ZIP}}</p>
  <p>
    <a href="{{unsubscribe_link}}">Unsubscribe</a> | 
    <a href="{{preferences_link}}">Email Preferences</a>
  </p>
</div>
```

## Tracking Opens and Clicks

### Pixel Tracking (Opens)

```html
<!-- Add to email HTML -->
<img src="https://yoursite.com/track/open/{{email_id}}/{{tracking_token}}" width="1" height="1" style="display:none">
```

### Link Tracking (Clicks)

```javascript
function trackableLink(originalUrl, emailId, token) {
  const trackingUrl = `https://yoursite.com/track/click/${emailId}/${token}`;
  const params = new URLSearchParams({
    redirect: originalUrl
  });
  return `${trackingUrl}?${params}`;
}

// Example
const ctaLink = trackableLink('https://yourproduct.com/signup', 'email123', 'token456');
```

## Drip Campaign Example

```javascript
// 5-email drip sequence
const dripSequence = [
  {
    day: 0,
    subject: 'Welcome to {{product}}!',
    template: 'welcome.html'
  },
  {
    day: 2,
    subject: 'How {{product}} saves you 5 hours/week',
    template: 'education.html'
  },
  {
    day: 5,
    subject: 'See {{product}} in action (2-min video)',
    template: 'demo.html'
  },
  {
    day: 7,
    subject: '100+ teams using {{product}} to {{benefit}}',
    template: 'social-proof.html'
  },
  {
    day: 10,
    subject: 'Ready to try {{product}}?',
    template: 'conversion.html'
  }
];

// Schedule emails
function scheduleDrip(email, signupDate) {
  dripSequence.forEach(step => {
    const sendDate = new Date(signupDate);
    sendDate.setDate(sendDate.getDate() + step.day);
    
    // Store in DB for cron job
    db.run(`
      INSERT INTO scheduled_emails (email, template, subject, send_at)
      VALUES (?, ?, ?, ?)
    `, [email, step.template, step.subject, sendDate]);
  });
}
```

## Best Practices

1. **Warm up your domain:**
   - Start with 10-20 emails/day
   - Gradually increase over 2-3 weeks
   - Maintain good sender reputation

2. **Segment your list:**
   - By industry, role, behavior
   - Send targeted messages
   - Higher engagement = better deliverability

3. **A/B test subject lines:**
   ```javascript
   const variants = [
     'Quick question about {{company}}',
     'Saw your {{company}} launch – congrats!',
     '{{company}} + {{product}} = ❤️?'
   ];
   // Randomly assign and track performance
   ```

4. **Monitor bounce rates:**
   - <2% = good
   - >5% = deliverability issues
   - Clean list regularly

5. **Avoid spam triggers:**
   - No ALL CAPS
   - No excessive !!!!!
   - No "FREE", "URGENT", "ACT NOW"
   - Balanced text/HTML ratio

## Testing Before Sending

```bash
# Send test email
node send-email.js "your-test@email.com" "test-campaign"

# Check spam score
# Use: https://www.mail-tester.com
# Send to their test address and get score

# Verify deliverability
# Use: https://mxtoolbox.com/emailhealth.aspx
```

## Troubleshooting

**Emails going to spam:**
- Set up SPF, DKIM, DMARC records
- Use authenticated domain
- Avoid spam trigger words
- Include plain text version
- Don't use URL shorteners

**Low open rates (<15%):**
- Test subject lines
- Send at optimal times (Tue-Thu, 10am-2pm)
- Clean inactive subscribers
- Segment list better

**High unsubscribe rate (>0.5%):**
- Content not relevant
- Emailing too frequently
- Better list targeting needed

**SMTP authentication failed:**
```bash
# Test credentials
curl -v --url "smtp://$CEOCLAW_SMTP_HOST:$CEOCLAW_SMTP_PORT" \
  --mail-from "$CEOCLAW_SMTP_USER" \
  --mail-rcpt "test@example.com" \
  --user "$CEOCLAW_SMTP_USER:$CEOCLAW_SMTP_PASS"
```

## Next Steps

After email outreach:
1. Track engagement in database
2. Follow up with interested replies
3. Refine messaging based on data
4. Monitor metrics (see `metrics.md`)
