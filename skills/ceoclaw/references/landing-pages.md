# Landing Page Generation Guide

Create production-ready landing pages with compelling copy, clean design, and conversion optimization.

## Overview

Generate responsive landing pages that:
- Communicate value proposition clearly
- Drive a single conversion action (signup, waitlist, purchase)
- Load fast and work on all devices
- Include analytics tracking
- Follow design best practices

## Tech Stack Options

### Option 1: Static HTML + Tailwind CSS (Fastest)
- Single file, no build step
- Deploy anywhere
- Perfect for MVPs
- ~5KB gzipped

### Option 2: Next.js + React (Modern)
- SEO optimized
- Easy to extend
- Vercel-optimized
- TypeScript support

### Option 3: Plain HTML + Inline CSS (Universal)
- Works everywhere
- No dependencies
- Easy to edit
- Portable

## GPT-4 Landing Page Prompt

```
You are an expert web developer and copywriter. Create a high-converting landing page.

PRODUCT DETAILS:
- Title: {idea.title}
- Problem: {idea.problem}
- Solution: {idea.solution}
- Target Market: {idea.target_market}
- Value Prop: {idea.value_proposition}

Generate a complete landing page as a single HTML file with:

1. **Structure:**
   - Hero section with headline + subheadline + CTA
   - Problem section (3 pain points)
   - Solution section (3 key benefits with icons)
   - How It Works (3-4 steps)
   - Social proof placeholder
   - Pricing/CTA section
   - FAQ (5 questions)
   - Footer

2. **Design:**
   - Use Tailwind CSS via CDN
   - Modern gradient backgrounds
   - Mobile-responsive (mobile-first)
   - Clean typography (Inter font)
   - Consistent spacing
   - Smooth animations on scroll

3. **Copy:**
   - Clear, benefit-focused headlines
   - Specific, not generic
   - Action-oriented CTAs
   - Address objections in FAQ

4. **Technical:**
   - Include Google Analytics (placeholder)
   - Email capture form with validation
   - Meta tags for SEO and social sharing
   - Favicon data URI
   - Loading performance optimized

Return ONLY the complete HTML file. Make it production-ready.
```

### Implementation

```bash
cd ~/.openclaw/ceoclaw

# Get idea from database
IDEA=$(sqlite3 ceoclaw.db <<SQL
SELECT json_object(
  'id', id,
  'title', title,
  'problem', description,
  'solution', solution,
  'target_market', target_market
)
FROM ideas
WHERE id = 1;  -- or specify idea ID
SQL
)

# Generate landing page with GPT-4
node <<'EOF'
const https = require('https');
const fs = require('fs');

const idea = JSON.parse(process.env.IDEA);

const prompt = `You are an expert web developer and copywriter.

Create a high-converting landing page for this product:

PRODUCT:
- Title: ${idea.title}
- Problem: ${idea.problem}
- Solution: ${idea.solution}
- Target: ${idea.target_market}

Generate a complete, production-ready HTML file with:

STRUCTURE:
1. Hero: Headline, subheadline, email signup form
2. Problem: 3 specific pain points with emojis
3. Solution: 3 key benefits with icons
4. How It Works: 3-4 simple steps
5. CTA: Strong call-to-action section
6. FAQ: 5 common questions
7. Footer: Simple footer with links

DESIGN:
- Use Tailwind CSS CDN
- Modern gradient backgrounds (blue/purple)
- Mobile-responsive
- Inter font from Google Fonts
- Smooth scroll animations
- Clean, minimal design

COPY REQUIREMENTS:
- Headlines: Clear, benefit-focused, <10 words
- Subheadlines: Specific value prop, 1 sentence
- Benefits: Concrete, not generic ("Save 5 hours/week" not "Save time")
- CTA: Action-oriented ("Start Free Trial" not "Submit")

TECHNICAL:
- Include <meta> tags for SEO
- OpenGraph/Twitter card meta
- Google Analytics placeholder
- Email form with JS validation
- Smooth scroll to sections
- Fast loading (inline critical CSS)

Return ONLY the complete HTML. No explanations. Make it production-ready.`;

const requestBody = JSON.stringify({
  model: 'gpt-4-turbo',
  messages: [{ role: 'user', content: prompt }],
  temperature: 0.7,
  max_tokens: 4000
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
    
    let html = result.choices[0].message.content;
    
    // Clean markdown code fence if present
    html = html.replace(/^```html\n/, '').replace(/\n```$/, '');
    
    // Write to file
    const filename = `landing-${idea.title.toLowerCase().replace(/\s+/g, '-')}.html`;
    fs.writeFileSync(filename, html);
    
    console.log(`✅ Landing page generated: ${filename}`);
    console.log(`   Preview: open ${filename}`);
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

## Static HTML Template Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ProductName - One-line value prop</title>
  
  <!-- SEO -->
  <meta name="description" content="Clear description with keywords">
  <meta name="keywords" content="keyword1, keyword2, keyword3">
  
  <!-- OpenGraph -->
  <meta property="og:title" content="ProductName">
  <meta property="og:description" content="Value proposition">
  <meta property="og:type" content="website">
  <meta property="og:url" content="https://yoursite.com">
  <meta property="og:image" content="https://yoursite.com/og-image.jpg">
  
  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="ProductName">
  <meta name="twitter:description" content="Value proposition">
  <meta name="twitter:image" content="https://yoursite.com/og-image.jpg">
  
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- Google Fonts -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
  
  <!-- Custom Styles -->
  <style>
    body { font-family: 'Inter', sans-serif; }
  </style>
  
  <!-- Analytics -->
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', 'G-XXXXXXXXXX');
  </script>
</head>
<body class="bg-gray-50">
  
  <!-- Hero Section -->
  <section class="bg-gradient-to-br from-blue-500 to-purple-600 text-white py-20">
    <div class="container mx-auto px-4 text-center">
      <h1 class="text-5xl font-bold mb-4">
        Compelling Headline Under 10 Words
      </h1>
      <p class="text-xl mb-8 max-w-2xl mx-auto">
        One-sentence value proposition that clearly explains the benefit.
      </p>
      
      <!-- Email Signup Form -->
      <form class="max-w-md mx-auto" onsubmit="handleSignup(event)">
        <div class="flex gap-2">
          <input 
            type="email" 
            placeholder="Enter your email" 
            required
            class="flex-1 px-4 py-3 rounded-lg text-gray-900"
          >
          <button 
            type="submit"
            class="px-8 py-3 bg-white text-blue-600 font-semibold rounded-lg hover:bg-gray-100 transition"
          >
            Get Started
          </button>
        </div>
      </form>
      
      <p class="mt-4 text-sm opacity-90">
        ✓ No credit card required  ✓ Free 14-day trial  ✓ Cancel anytime
      </p>
    </div>
  </section>
  
  <!-- Problem Section -->
  <section class="py-16">
    <div class="container mx-auto px-4">
      <h2 class="text-3xl font-bold text-center mb-12">
        Tired of These Problems?
      </h2>
      <div class="grid md:grid-cols-3 gap-8 max-w-4xl mx-auto">
        <div class="text-center">
          <div class="text-4xl mb-4">😤</div>
          <h3 class="font-semibold mb-2">Specific Problem #1</h3>
          <p class="text-gray-600">Brief description of pain point</p>
        </div>
        <div class="text-center">
          <div class="text-4xl mb-4">⏰</div>
          <h3 class="font-semibold mb-2">Specific Problem #2</h3>
          <p class="text-gray-600">Brief description of pain point</p>
        </div>
        <div class="text-center">
          <div class="text-4xl mb-4">💸</div>
          <h3 class="font-semibold mb-2">Specific Problem #3</h3>
          <p class="text-gray-600">Brief description of pain point</p>
        </div>
      </div>
    </div>
  </section>
  
  <!-- Solution Section -->
  <section class="py-16 bg-white">
    <div class="container mx-auto px-4">
      <h2 class="text-3xl font-bold text-center mb-12">
        Here's How We Solve It
      </h2>
      <div class="grid md:grid-cols-3 gap-8 max-w-4xl mx-auto">
        <div>
          <div class="text-blue-600 text-3xl mb-4">✓</div>
          <h3 class="font-semibold mb-2">Key Benefit #1</h3>
          <p class="text-gray-600">Specific, measurable outcome</p>
        </div>
        <div>
          <div class="text-blue-600 text-3xl mb-4">✓</div>
          <h3 class="font-semibold mb-2">Key Benefit #2</h3>
          <p class="text-gray-600">Specific, measurable outcome</p>
        </div>
        <div>
          <div class="text-blue-600 text-3xl mb-4">✓</div>
          <h3 class="font-semibold mb-2">Key Benefit #3</h3>
          <p class="text-gray-600">Specific, measurable outcome</p>
        </div>
      </div>
    </div>
  </section>
  
  <!-- How It Works -->
  <section class="py-16">
    <div class="container mx-auto px-4 max-w-3xl">
      <h2 class="text-3xl font-bold text-center mb-12">
        How It Works
      </h2>
      <div class="space-y-8">
        <div class="flex items-start gap-4">
          <div class="flex-shrink-0 w-12 h-12 bg-blue-600 text-white rounded-full flex items-center justify-center font-bold">
            1
          </div>
          <div>
            <h3 class="font-semibold mb-1">Simple Step One</h3>
            <p class="text-gray-600">Brief explanation</p>
          </div>
        </div>
        <div class="flex items-start gap-4">
          <div class="flex-shrink-0 w-12 h-12 bg-blue-600 text-white rounded-full flex items-center justify-center font-bold">
            2
          </div>
          <div>
            <h3 class="font-semibold mb-1">Simple Step Two</h3>
            <p class="text-gray-600">Brief explanation</p>
          </div>
        </div>
        <div class="flex items-start gap-4">
          <div class="flex-shrink-0 w-12 h-12 bg-blue-600 text-white rounded-full flex items-center justify-center font-bold">
            3
          </div>
          <div>
            <h3 class="font-semibold mb-1">Simple Step Three</h3>
            <p class="text-gray-600">Brief explanation</p>
          </div>
        </div>
      </div>
    </div>
  </section>
  
  <!-- CTA Section -->
  <section class="py-20 bg-gradient-to-br from-blue-500 to-purple-600 text-white text-center">
    <div class="container mx-auto px-4">
      <h2 class="text-4xl font-bold mb-4">
        Ready to Get Started?
      </h2>
      <p class="text-xl mb-8 max-w-2xl mx-auto">
        Join thousands of users already saving time and money.
      </p>
      <button class="px-12 py-4 bg-white text-blue-600 font-semibold text-lg rounded-lg hover:bg-gray-100 transition">
        Start Your Free Trial
      </button>
    </div>
  </section>
  
  <!-- FAQ -->
  <section class="py-16">
    <div class="container mx-auto px-4 max-w-3xl">
      <h2 class="text-3xl font-bold text-center mb-12">
        Frequently Asked Questions
      </h2>
      <div class="space-y-6">
        <details class="bg-white p-6 rounded-lg shadow">
          <summary class="font-semibold cursor-pointer">
            Question about pricing?
          </summary>
          <p class="mt-4 text-gray-600">
            Clear, honest answer.
          </p>
        </details>
        <!-- More FAQs... -->
      </div>
    </div>
  </section>
  
  <!-- Footer -->
  <footer class="bg-gray-900 text-white py-8">
    <div class="container mx-auto px-4 text-center">
      <p>&copy; 2026 ProductName. All rights reserved.</p>
      <div class="mt-4 space-x-4">
        <a href="#" class="hover:text-blue-400">Privacy</a>
        <a href="#" class="hover:text-blue-400">Terms</a>
        <a href="#" class="hover:text-blue-400">Contact</a>
      </div>
    </div>
  </footer>
  
  <!-- JavaScript -->
  <script>
    function handleSignup(e) {
      e.preventDefault();
      const email = e.target.querySelector('input[type="email"]').value;
      
      // Send to your backend or email service
      fetch('/api/signup', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email })
      })
      .then(res => res.json())
      .then(data => {
        alert('Thanks for signing up! Check your email.');
        e.target.reset();
      })
      .catch(err => {
        console.error('Signup error:', err);
        alert('Error signing up. Please try again.');
      });
    }
    
    // Smooth scroll
    document.querySelectorAll('a[href^="#"]').forEach(anchor => {
      anchor.addEventListener('click', function (e) {
        e.preventDefault();
        const target = document.querySelector(this.getAttribute('href'));
        if (target) {
          target.scrollIntoView({ behavior: 'smooth' });
        }
      });
    });
  </script>
</body>
</html>
```

## Copy Guidelines

### Headlines (Hero)
✅ Good:
- "Ship Landing Pages in 5 Minutes"
- "Email Marketing That Actually Converts"
- "Turn Reddit Comments Into Startup Ideas"

❌ Bad:
- "The Best Tool Ever" (generic)
- "Revolutionary Platform" (vague)
- "Innovative Solution" (meaningless)

### Value Props (Subheadline)
✅ Good:
- "Generate production-ready landing pages with GPT-4 and deploy in seconds"
- "Send 10x more cold emails without getting flagged as spam"
- "Track every mention of your product across 50+ platforms in one dashboard"

❌ Bad:
- "Make your life easier" (how?)
- "The future of productivity" (hype)
- "All-in-one solution" (unclear)

### Benefits (Features Section)
✅ Good:
- "Save 5 hours per week on manual research"
- "Get 30% higher open rates with AI personalization"
- "Deploy in 60 seconds with zero config"

❌ Bad:
- "Fast and easy" (relative)
- "Best in class" (claim)
- "Powerful features" (vague)

### CTAs
✅ Good:
- "Start Free Trial"
- "Get Early Access"
- "Join Waitlist"
- "See How It Works"

❌ Bad:
- "Submit"
- "Click Here"
- "Learn More"

## Optimization Checklist

### Performance
- [ ] Inline critical CSS
- [ ] Lazy load images below fold
- [ ] Minify HTML/CSS/JS
- [ ] Use CDNs for libraries
- [ ] Compress images (WebP format)
- [ ] Total size <200KB

### SEO
- [ ] Semantic HTML (h1, h2, section, etc.)
- [ ] Meta description <160 chars
- [ ] OpenGraph tags
- [ ] Twitter Card tags
- [ ] Alt text on images
- [ ] Mobile-friendly (responsive)

### Conversion
- [ ] Single clear CTA above fold
- [ ] Email capture form prominent
- [ ] Social proof (testimonials/logos)
- [ ] Address objections (FAQ)
- [ ] Trust signals (security, privacy)
- [ ] Fast loading (<3s)

### Design
- [ ] Mobile-first responsive
- [ ] Consistent spacing/typography
- [ ] High contrast text (WCAG AA)
- [ ] Visual hierarchy clear
- [ ] CTA buttons stand out
- [ ] Clean, uncluttered layout

## Testing Before Deploy

```bash
# Validate HTML
curl -F "out=gnu" -F "file=@landing.html" \
  https://validator.w3.org/nu/

# Check mobile responsiveness
open -a "Google Chrome" --args --user-agent="Mozilla/5.0 (iPhone; CPU iPhone OS 15_0 like Mac OS X)" landing.html

# Test load time
curl -w "@curl-format.txt" -o /dev/null -s http://localhost:8000/landing.html

# curl-format.txt:
# time_total: %{time_total}s
# size_download: %{size_download} bytes
```

## Next Steps

After generating landing page:
1. Review and edit copy for clarity
2. Test on mobile devices
3. Deploy to production (see `deployment.md`)
4. Set up analytics tracking
5. A/B test variations

## Troubleshooting

**GPT-4 returns incomplete HTML:**
- Increase `max_tokens` to 4000-8000
- Chain prompts: "Continue from where you left off"
- Generate sections separately and combine

**Form doesn't submit:**
- Add proper backend endpoint
- Use formspree.io or similar for quick setup
- Add client-side validation

**Styling broken:**
- Check Tailwind CDN loads
- Verify no conflicting CSS
- Test in different browsers
