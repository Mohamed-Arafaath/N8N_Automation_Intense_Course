# 📘 PHASE 2: Infinite Use Cases - Complete Cheat Sheet

> **Production-Ready Workflow Patterns** - Templates for Marketing, Sales, and Operations automation.

---

## 📱 1. Marketing Automation

### SEO Blog Factory - Complete Workflow

```
Trigger: Schedule (Daily 9 AM)
↓
1. KEYWORD RESEARCH
   Node: HTTP Request → SerpApi
   Endpoint: /search?engine=google_trends
   Extract: Rising keywords with volume > 1000

2. COMPETITOR ANALYSIS
   Node: HTTP Request → Firecrawl
   Action: Scrape top 10 results
   Extract: Word count, headers, topics

3. OUTLINE GENERATION
   Node: OpenAI Chat (Claude 3.5 Sonnet)
   Prompt:
   """
   Analyze these top 10 articles for "{keyword}":
   {competitor_data}
   
   Create SEO outline that:
   - Covers all competitor topics
   - Fills content gaps
   - 2000+ words target
   - H2/H3 structure
   """

4. CONTENT GENERATION
   Node: OpenAI Chat (GPT-4o)
   Prompt:
   """
   Write comprehensive article using this outline:
   {outline}
   
   Requirements:
   - Engaging introduction
   - Examples and case studies
   - Internal links: {{$json.internal_links}}
   - Strong CTA
   - Keyword density: 1-2%
   """

5. IMAGE GENERATION
   Node: OpenAI DALL-E
   Prompt: "Professional blog featured image for: {title}"
   Size: 1200x630

6. SEO OPTIMIZATION
   Node: Code Node
   ```javascript
   const content = $input.first().json.article;
   const keyword = $input.first().json.keyword;
   
   // Calculate keyword density
   const wordCount = content.split(' ').length;
   const keywordCount = (content.match(new RegExp(keyword, 'gi')) || []).length;
   const density = (keywordCount / wordCount) * 100;
   
   // Generate meta description
   const metaDesc = content.substring(0, 160);
   
   return [{
     json: {
       content,
       keyword_density: density.toFixed(2),
       meta_description: metaDesc,
       title: `${keyword} | Complete Guide 2025`
     }
   }];
   ```

7. PUBLISH TO WORDPRESS
   Node: WordPress
   Operation: Create Post
   Status: Draft (or Publish)
   Categories: {auto_detected}

8. SOCIAL DISTRIBUTION
   - LinkedIn: Professional summary
   - Twitter: Thread (5-7 tweets)
   - Facebook: Engagement post
```

**Time Saved:** 4 hours → 15 minutes

---

### Social Media Command Center

#### Platform-Specific Content Adaptation

**LinkedIn Prompt:**
```
Convert this article into LinkedIn carousel (10 slides):

Article: {article_content}

Requirements:
- Slide 1: Hook (attention-grabbing statement)
- Slides 2-9: Key insights (1 per slide)
- Slide 10: CTA
- Professional tone
- Include statistics
- Each slide: Headline + 2-3 bullets
```

**Twitter Thread Prompt:**
```
Create Twitter thread:

Article: {article_content}

Format:
1/ Hook tweet (must stop scroll)
2/ Context
3-7/ Key points (1 insight per tweet)
8/ Summary + CTA

Rules:
- Max 280 chars per tweet
- Use line breaks
- Add relevant emoji
- Include hashtags (max 3)
```

**Instagram Reel Script:**
```
Create 30-second Instagram Reel script:

Topic: {article_topic}

Format:
0-3s: Hook (question or bold statement)
3-10s: Problem
10-25s: Solution (3 tips)
25-30s: CTA

Include:
- Text overlay suggestions
- B-roll ideas
- Hashtag strategy (15-20 tags)
```

#### Optimal Posting Times

```javascript
// Code Node - Determine best posting time
const platform = $input.first().json.platform;
const timezone = 'America/New_York';

const schedules = {
  linkedin: {
    days: [2, 3, 4], // Tue-Thu
    hour: 10
  },
  twitter: {
    days: [1, 2, 3, 4, 5], // Mon-Fri
    hour: 12
  },
  instagram: {
    days: [0, 1, 2, 3, 4, 5, 6], // Every day
    hour: 19
  },
  tiktok: {
    days: [0, 1, 2, 3, 4, 5, 6],
    hour: 21
  },
  facebook: {
    days: [3, 4, 5], // Wed-Fri
    hour: 13
  }
};

const schedule = schedules[platform];
const postTime = new Date();
postTime.setHours(schedule.hour, 0, 0, 0);

return [{ json: { schedule_time: postTime.toISOString() } }];
```

---

### YouTube Shorts Automation

```
1. TREND MONITORING
   Sources:
   - Reddit Hot Posts (r/technology, r/business)
   - Twitter Trending Topics
   Filter: Engagement > 10k

2. SCRIPT GENERATION
   Prompt:
   """
   Create 30-second YouTube Shorts script about: {topic}
   
   Structure:
   0-3s: HOOK (must grab attention)
   3-12s: Setup (1 key point)
   12-24s: Payoff (surprising fact)
   24-30s: CTA (like & follow)
   
   Tone: Conversational, energetic
   """

3. VOICEOVER
   Node: ElevenLabs
   Voice: Professional Male/Female
   Settings:
   - Stability: 0.5
   - Similarity: 0.75
   - Style: 0.3

4. VIDEO GENERATION
   Node: HTTP Request → Runway ML API
   Prompt: "{visual_description}"
   Aspect: 9:16 (vertical)
   Duration: 30s

5. CAPTION GENERATION
   Style: Bold yellow text
   Position: Center
   Animation: Word-by-word

6. THUMBNAIL
   Node: DALL-E
   Prompt: "Eye-catching YouTube thumbnail: {topic}"
   Size: 1280x720
   Style: Vibrant, clickable

7. UPLOAD
   Node: YouTube API
   Set as Short: Yes
   Privacy: Public
   Title: "{keyword_optimized_title}"
   Description: (150 words, keywords in first 2 sentences)
   Tags: {15_relevant_tags}
```

---

## 💰 2. Sales Intelligence & Lead Generation

### The Hunter Agent - Google Maps Lead Scraper

```
1. INPUT PARAMETERS
   {
     "industry": "Dentists",
     "location": "Miami, FL",
     "quantity": 100
   }

2. GOOGLE MAPS SCRAPING
   Node: HTTP Request → SerpApi
   Query: "{industry} in {location}"
   Results: {quantity}
   
   Extract:
   - Business name
   - Address  
   - Phone
   - Website
   - Rating
   - Review count

3. EMAIL ENRICHMENT
   Node: HTTP Request → Apollo.io
   Input: Business name + domain
   Output: Decision-maker emails
   
   Alternative: Hunter.io
   Endpoint: /v2/domain-search
   Domain: {business_domain}

4. WEBSITE ANALYSIS
   Node: HTTP Request → Firecrawl
   Action: Extract full content
   
   Then: AI Analysis
   ```javascript
   Prompt:
   """
   Analyze this dental practice website:
   {website_content}
   
   Identify:
   1. Services offered
   2. Tech stack (online booking? chatbot?)
   3. Pain points (outdated design, missing features)
   4. Value proposition
   5. Recent news/updates
   """
   ```

5. PERSONALIZATION ENGINE
   Prompt:
   """
   Write cold email to {business_name}:
   
   Context:
   - Industry: {industry}
   - Location: {city}
   - Website: {url}
   - Pain point: {identified_pain}
   
   Email structure:
   - Subject: Reference specific pain point
   - Line 1: Personalized observation
   - Line 2-3: Solution (our service)
   - Line 4: Social proof (similar client)
   - CTA: 15-min call
   - Max 100 words
   - Professional but friendly tone
   """

6. SEND EMAIL
   Node: Gmail / SendGrid
   Delay: 2-5 min between sends (avoid spam)
   Track: Opens, clicks

7. CRM INTEGRATION
   Node: HubSpot
   Action: Create Contact
   Properties:
   - Lead Source: "n8n Automation"
   - Lead Score: {calculated_score}
   - Next Action: "Email sent - await reply"
```

**Lead Scoring Formula:**
```javascript
let score = 0;

// Company size (if available)
if (employees > 50) score += 30;
else if (employees > 10) score += 15;

// Website quality
if (hasOnlineBooking) score += 20;
else score += 10; // Opportunity to sell

// Reviews
if (rating >= 4.5 && reviewCount > 50) score += 25;

// Location (affluent area check)
if (affluent_zip_codes.includes(zip)) score += 15;

// Engagement potential
if (recentlyHiring) score += 10;

return score; // 0-100
```

---

### BANT Qualification Bot

```
1. TRIGGER: New lead (webhook/form)

2. INITIAL EMAIL (Day 0)
   Subject: "Quick question about {pain_point}"
   Body:
   """
   Hi {first_name},
   
   I noticed {company} might benefit from {solution}.
   
   Quick question: Are you currently looking to solve {pain_point}?
   
   If so, I'd love to share how we helped {similar_company} achieve {result}.
   
   Worth a quick chat?
   
   Best,
   {your_name}
   ```

3. WAIT FOR REPLY (3 days)

4. IF NO REPLY → FOLLOW-UP
   Subject: "Re: Quick question"
   Body:
   """
   {first_name},
   
   Following up on my message below. 
   
   I have a 2-minute video showing exactly how this works.
   
   Want me to send it?
   """

5. IF REPLY → QUALIFICATION SEQUENCE

   EMAIL 2 (Budget - B)
   """
   Great to hear from you!
   
   To make sure we're a good fit, what's your budget range?
   
   a) Under $5,000
   b) $5,000 - $20,000
   c) $20,000+
   """

   EMAIL 3 (Authority - A)
   """
   Perfect. One more question:
   
   Are you the primary decision-maker, or will others be involved?
   """

   EMAIL 4 (Timeline - T)
   """
   Got it. When are you looking to implement?
   
   a) ASAP (this month)
   b) Next 1-3 months
   c) Just exploring options
   ```

6. AI SCORING
   ```javascript
   let leadScore = 0;
   
   // Budget (30 points)
   if (budget === 'c') leadScore += 30;
   else if (budget === 'b') leadScore += 20;
   else leadScore += 10;
   
   // Authority (25 points)
   if (isDecisionMaker) leadScore += 25;
   else leadScore += 12;
   
   // Timeline (25 points)
   if (timeline === 'a') leadScore += 25;
   else if (timeline === 'b') leadScore += 15;
   else leadScore += 5;
   
   // Need (20 points) - based on initial response
   if (strongNeed) leadScore += 20;
   else leadScore += 10;
   
   return leadScore;
   ```

7. ROUTING
   ```
   IF score >= 80 (Hot Lead):
   → Slack alert to sales rep
   → Send Calendly link immediately
   → Flag as "Priority" in CRM
   
   IF score 50-79 (Warm Lead):
   → Add to weekly nurture campaign
   → Educational content + case studies
   
   IF score < 50 (Cold Lead):
   → Long-term nurture (monthly)
   → Requalify in 6 months
   ```
```

---

### Meeting Prep Assistant

```
TRIGGER: 15 min before calendar event

1. EXTRACT ATTENDEE INFO
   Source: Google Calendar event
   Get: Attendee emails

2. LINKEDIN SCRAPING
   For each attendee:
   - Current role
   - Company
   - Recent posts
   - Career history

3. COMPANY INTELLIGENCE
   Sources:
   - Crunchbase (funding, investors)
   - Google News (recent news)
   - SimilarWeb (traffic data)
   - Clearbit (tech stack)

4. GENERATE BRIEFING
   Prompt:
   """
   Create meeting prep document:
   
   Attendees: {attendee_data}
   Company: {company_data}
   
   Include:
   1. Key talking points (3-5)
   2. Potential pain points (what they need)
   3. Conversation starters
   4. Company context
   5. Recommended approach
   
   Format: Professional, concise
   """

5. DELIVER VIA SLACK
   Message:
   """
   🎯 Meeting in 15 minutes with {company}
   
   **Attendees:**
   - {name} ({role})
   
   **talking Points:**
   {talking_points}
   
   **Recent News:**
   {news}
   
   **Recommendation:**
   {strategy}
   
   📎 Full brief: {google_docs_link}
   ```
```

---

## 🏢 3. Operations & Finance Automation

### Invoice Processing

```
1. EMAIL TRIGGER
   Monitor: invoices@company.com
   Filter: Has PDF attachment

2. EXTRACT DATA
   Node: PDF Extract
   Fields:
   - Invoice number
   - Vendor name
   - Amount
   - Due date
   - Line items

3. VALIDATE
   ```javascript
   // Code Node
   const invoice = $input.first().json;
   
   // Check for PO match
   const hasPO = invoice.items.every(item => item.po_number);
   
   // Check amount threshold
   const requiresApproval = invoice.total > 5000;
   
   // Check vendor
   const isApprovedVendor = approvedVendors.includes(invoice.vendor);
   
   return [{
     json: {
       ...invoice,
       needs_approval: requiresApproval,
       auto_approve: hasPO && isApprovedVendor && !requiresApproval
     }
   }];
   ```

4. ROUTING
   IF auto_approve:
   → Pay via Bill.com API
   → Update QuickBooks
   → Archive in Google Drive
   
   IF needs_approval:
   → Send to Slack approval channel
   → Include all details
   → One-click approve button

5. RECORD KEEPING
   - Log in Airtable
   - Update budget tracking
   - Generate monthly report
```

### Expense Categorization

```
1. BANK FEED
   Source: Plaid API / Bank API
   Trigger: New transaction

2. AI CATEGORIZATION
   Prompt:
   """
   Categorize this expense:
   
   Merchant: {merchant_name}
   Amount: {amount}
   Description: {description}
   
   Categories:
   - Office Supplies
   - Software/SaaS
   - Marketing
   - Travel
   - Meals & Entertainment
   - Other
   
   Return: Category name
   """

3. VALIDATE & LEARN
   - Store categorization
   - Learn from corrections
   - Build custom rules

4. SYNC TO QUICKBOOKS
   Create: Expense record
   Category: {ai_category}
   Attach: Receipt (if available)
```

---

## 🎯 Industry-Specific Quick Patterns

### Real Estate
```
- Lead from Zillow → CRM → Email sequence
- Property listing → Social posts (all platforms)
- Contract signing → Automated welcome package
```

### Healthcare
```
- Appointment booking → Reminder sequence
- Insurance verification (API integration)
- Patient feedback → Review request automation
```

### Legal
```
- Case intake form → Document generation
- Contract review → AI-powered analysis
- Client billing → Invoice automation
```

### E-commerce
```
- Order placed → Fulfillment + tracking
- Abandoned cart → Recovery sequence
- Low stock → Reorder alert
```

---

**💡 Pro Tips:**
- Always test with 1-2 items first
- Use delays between API calls (rate limiting)
- Include error handling for all external APIs
- Log everything for debugging
- Monitor costs (AI API usage)
