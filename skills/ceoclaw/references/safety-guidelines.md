# Safety Guidelines for CEOClaw

CEOClaw performs real-world actions that can have consequences. Follow these guidelines to use it responsibly.

## Core Principles

### 1. Human Oversight

**Always review before executing**:
- Deployment URLs and content
- Email recipients and messages
- Social media posts
- Marketing content

### 2. Gradual Scaling

Start small:
1. **Dry Run**: Test with safety flags enabled
2. **Manual Review**: Approve each action
3. **Limited Scope**: Small recipient lists, low budgets
4. **Monitor**: Watch metrics and feedback
5. **Scale Up**: Only after validating results

### 3. Compliance First

Before launching:
- Review applicable laws (CAN-SPAM, GDPR, etc.)
- Check platform Terms of Service
- Consult legal counsel for regulated industries
- Implement opt-out mechanisms
- Maintain privacy policies

## Deployment Safety

### Before Deploying

- [ ] Review landing page HTML/content
- [ ] Check for sensitive information (API keys, passwords)
- [ ] Verify no trademark violations
- [ ] Test locally first
- [ ] Ensure appropriate disclaimers
- [ ] Validate accessibility compliance

### Deployment Checklist

```json
{
  "approval": {
    "requireForDeployments": true  // Keep enabled until vetted
  }
}
```

### Post-Deployment

- Monitor for abuse reports
- Check analytics for unusual patterns
- Have rollback plan ready
- Respond to user feedback promptly

## Email Outreach Safety

### Legal Requirements

**CAN-SPAM Act (US)**:
- Include physical address
- Clear opt-out mechanism
- Accurate subject lines
- Identify as advertisement

**GDPR (EU)**:
- Explicit consent required
- Right to erasure
- Data protection measures
- Privacy policy link

### Best Practices

```json
{
  "approval": {
    "requireForEmails": true  // STRONGLY RECOMMENDED
  }
}
```

**Do**:
- Personalize messages
- Provide value upfront
- Make opt-out easy
- Honor unsubscribe requests immediately
- Keep lists clean (remove bounces)

**Don't**:
- Buy email lists
- Send to unverified addresses
- Mislead about sender identity
- Spam or harass
- Ignore opt-out requests

### Rate Limiting

```javascript
// Built into email-outreach tool
await delay(1000); // 1 second between emails
```

Never:
- Send more than 50 emails/batch
- Exceed platform daily limits
- Retry failed sends aggressively

## Social Media Safety

### Platform Terms of Service

**Twitter/X**:
- No automation for engagement
- Real person must be in control
- Disclose bot/AI usage

**LinkedIn**:
- Commercial use requires permission
- No automated messaging to connections
- Professional content only

**Reddit**:
- Must add value to community
- No spam or self-promotion
- Disclose affiliation
- Respect subreddit rules

### Best Practices

```json
{
  "approval": {
    "requireForPosts": true  // Always review first
  }
}
```

**Do**:
- Engage authentically
- Add value to discussions
- Disclose AI-generated content
- Respond to feedback
- Follow community guidelines

**Don't**:
- Cross-post spam
- Manipulate votes/engagement
- Create fake accounts
- Mislead about product
- Ignore negative feedback

## Budget & Cost Safety

### Set Hard Limits

```json
{
  "budget": {
    "maxMonthlySpend": 100  // Start low
  }
}
```

### Monitor Spending

Track costs across:
- OpenAI API ($)
- Vercel deployments (usually free)
- Email sending ($)
- Analytics (usually free)

### Cost Optimization

- Use GPT-3.5-turbo for drafts
- Cache AI responses
- Limit market research iterations
- Use free analytics tiers

## Data Privacy

### User Data

**Collect Only**:
- Email addresses (with consent)
- Basic analytics (anonymized)
- Signup information

**Never Collect**:
- Passwords (use OAuth)
- Payment details (use Stripe/processors)
- Sensitive personal info
- Unnecessary tracking data

### Storage

- Encrypt sensitive data
- Use secure environment variables
- Regular backups
- Secure deletion when requested

### Database Security

SQLite database location: `~/.ceoclaw/data.db`

```bash
# Set restrictive permissions
chmod 600 ~/.ceoclaw/data.db
```

## Error Handling

### Graceful Failures

All tools implement:
- Try-catch blocks
- Meaningful error messages
- Continue on non-critical errors
- Log for debugging

### Rollback Strategy

If deployment goes wrong:

```bash
# Vercel rollback
vercel rollback [deployment-url]

# Email retraction
# (Not possible - review BEFORE sending)
```

## Monitoring & Alerts

### Track These Metrics

- Deployment failures
- Email bounce rates
- Spam complaints
- API errors
- Budget overruns

### Alert Thresholds

```javascript
if (bounceRate > 0.1) {
  // Stop email campaign
  // Review recipient list
}

if (spamComplaints > 0) {
  // Immediate halt
  // Review content
  // Contact recipients
}
```

## Incident Response

If something goes wrong:

1. **Immediate Actions**
   - Stop all automated actions
   - Assess scope of issue
   - Document what happened

2. **Communication**
   - Notify affected users
   - Issue public statement if needed
   - Be transparent about AI usage

3. **Remediation**
   - Fix root cause
   - Compensate if appropriate
   - Update safety measures

4. **Prevention**
   - Review processes
   - Update guidelines
   - Improve approval gates

## Testing Safely

### Dry Run Mode

```bash
# Test without real actions
CEOCLAW_DRY_RUN=1 npm run ceoclaw
```

### Staging Environment

Before production:
1. Deploy to staging URL
2. Send test emails to yourself
3. Post to private groups
4. Verify all integrations

### Test Dataset

Use synthetic data:
```json
{
  "recipients": [
    {"email": "test1@example.com", "name": "Test User 1"},
    {"email": "your-own@email.com", "name": "You"}
  ]
}
```

## Red Flags to Watch

Stop immediately if:
- High bounce/complaint rates
- User reports of harassment
- Platform ToS violations
- Budget overruns
- Security incidents
- Legal notices

## When NOT to Use CEOClaw

**Inappropriate use cases**:
- Healthcare services (HIPAA requirements)
- Financial services (SEC regulations)
- Legal services (ethics rules)
- Adult content
- Anything requiring licensed professionals
- Highly regulated industries

**Consult experts first!**

## Ethical AI Use

### Transparency

- Disclose AI-generated content
- Be clear about automation
- Don't impersonate humans
- Credit sources appropriately

### Fairness

- Avoid discriminatory targeting
- Ensure accessible content
- Respect user autonomy
- Honor user choices

### Accountability

- Take responsibility for outputs
- Respond to concerns promptly
- Maintain audit logs
- Accept feedback

## Approval Workflow

### Recommended Settings

**Development**:
```json
{
  "approval": {
    "requireForDeployments": true,
    "requireForEmails": true,
    "requireForPosts": true
  }
}
```

**Production** (after validation):
```json
{
  "approval": {
    "requireForDeployments": false,  // If confident
    "requireForEmails": true,        // Always review
    "requireForPosts": true          // Always review
  }
}
```

### Review Process

1. **Review Content**: Check for quality, accuracy, appropriateness
2. **Verify Recipients**: Ensure targeting is correct
3. **Test First**: Send to yourself or small test group
4. **Monitor Results**: Watch metrics and feedback
5. **Iterate**: Improve based on learnings

## Resources

### Legal
- CAN-SPAM: https://www.ftc.gov/tips-advice/business-center/guidance/can-spam-act-compliance-guide-business
- GDPR: https://gdpr.eu/
- CCPA: https://oag.ca.gov/privacy/ccpa

### Platform Guidelines
- Twitter: https://help.twitter.com/en/rules-and-policies/twitter-automation
- Reddit: https://www.redditinc.com/policies/content-policy
- LinkedIn: https://www.linkedin.com/legal/user-agreement

### Ethics
- AI Ethics Guidelines: https://www.unesco.org/en/artificial-intelligence/recommendation-ethics
- Tech Ethics: https://ethicalos.org/

## Summary

✅ **Safe CEOClaw Usage**:
- Always review before production
- Start with dry runs
- Enable all approval gates
- Monitor results closely
- Respect laws and ToS
- Be transparent about AI
- Accept responsibility

❌ **Unsafe CEOClaw Usage**:
- Blindly auto-running everything
- Ignoring rate limits
- Spamming users
- Misleading content
- Violating platform rules
- Avoiding accountability

---

**Remember**: CEOClaw is a powerful tool. Use it responsibly and ethically. When in doubt, ask for human review.
