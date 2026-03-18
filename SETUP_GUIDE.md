# Marvice - Social Media Lead Generation & Engagement Campaign

## Purpose

This n8n workflow is designed specifically for **Marvice** to:
- **Generate leads** by positioning Marvice as a thought leader across multiple niches
- **Drive engagement** (likes, comments, shares, saves, follows) on every post
- **Build brand authority** in AI, Digital Marketing, Web Development & Corporate Gifting
- **Auto-post** branded content with lead-gen CTAs to LinkedIn, Instagram, Facebook & Pinterest

Every post mentions **Marvice** and ends with a **call-to-action** driving DMs, page visits, or consultations.

---

## Workflow Pipeline

```
Schedule (Mon-Fri 8AM) ──┐
Manual Trigger ──────────┤
                         ▼
    ┌─── Google Trends ───────────────────────┐
    ├─── AI & Tech News (RSS) ────────────────┤
    ├─── Digital Marketing News (RSS) ────────┤── Aggregate Multi-Niche Trends
    ├─── Web Development News (RSS) ──────────┤
    └─── Corporate Gifting News (RSS) ────────┘
                         ▼
         Perplexity AI (pick highest lead-gen potential topic)
                         ▼
         OpenAI GPT-4o (Marvice-branded platform content + CTAs)
                         ▼
         DALL-E 3 (Marvice-branded poster in blue/white/gold)
                         ▼
         ImgBB (host image)
                         ▼
    ┌────────┬───────────┬──────────┐
    ▼        ▼           ▼          ▼
 LinkedIn  Instagram  Facebook  Pinterest
 (thought   (visual    (engage   (SEO pins
  leader)    + CTA)     + CTA)    + leads)
    └────────┴───────────┴──────────┘
                         ▼
         Google Sheets (lead tracking) → Slack (team notify)
```

---

## How Marvice Gets Leads from Each Platform

| Platform | Lead-Gen Strategy | CTA Style |
|----------|------------------|-----------|
| **LinkedIn** | Thought-leadership posts with data + actionable insights | "Follow Marvice for daily insights" / "DM us to discuss your project" |
| **Instagram** | Scroll-stopping hooks + value-packed tips + branded poster | "Save this for later" / "Follow @marvice" / "Link in bio for free consultation" |
| **Facebook** | Relatable pain points + engagement questions | "Comment GROWTH if you want a free audit" / "Tag someone who needs this" |
| **Pinterest** | SEO-rich pins with keywords for long-term discoverability | "Marvice | [Topic] guide" / drives traffic to Marvice page |

---

## Content Niches & Marvice Service Mapping

| Niche | Marvice Service | Lead Angle |
|-------|----------------|------------|
| **AI & Technology** | AI solutions, automation, AI consulting | "Let Marvice automate your business with AI" |
| **Digital Marketing** | SEO, social media marketing, PPC, analytics | "Marvice can 3x your marketing ROI" |
| **Web Development** | Websites, web apps, frontend/backend dev | "Need a website? Marvice builds conversion-ready sites" |
| **Corporate Gifting** | Business gifts, branded merch, employee rewards | "Marvice handles corporate gifting at scale" |
| **Marketing Updates** | Branding, growth strategy, viral campaigns | "Marvice is your growth partner" |

---

## Workflow Stages

| Stage | Node(s) | Purpose |
|-------|---------|---------|
| 1. Trigger | Schedule (Mon-Fri 8AM) or Manual | Initiates the daily campaign |
| 2. Research | Google Trends + 4 Niche RSS Feeds | Fetches trends across 5 content niches |
| 3. Aggregate | Code Node | Merges all sources into unified topic list |
| 4. Analysis | Perplexity AI | Selects topic with highest lead-gen potential for Marvice |
| 5. Content | OpenAI GPT-4o (HTTP Request) | Generates Marvice-branded posts with CTAs |
| 6. Design | DALL-E 3 (HTTP Request) | Creates Marvice-branded HD poster (blue/white/gold) |
| 7. Host | ImgBB | Hosts poster with persistent URL |
| 8. Publish | LinkedIn, Instagram, Facebook, Pinterest | Posts in parallel |
| 9. Track | Google Sheets + Slack | Logs campaign with lead tracking columns |

---

## Prerequisites & Credentials

### Required API Keys

| Service | Purpose | How to Get |
|---------|---------|------------|
| **OpenAI API** | Content generation + DALL-E poster (via HTTP Request) | https://platform.openai.com/api-keys |
| **Perplexity API** | Trend analysis & topic selection | https://docs.perplexity.ai/ |
| **ImgBB API** | Image hosting | https://api.imgbb.com/ |
| **LinkedIn OAuth2** | LinkedIn posting | LinkedIn Developer Portal |
| **Facebook/Instagram** | FB & IG posting | Meta Developer Portal |
| **Pinterest API** | Pinterest pinning | Pinterest Developer Portal |
| **Google Sheets OAuth2** | Lead tracking & campaign log | Google Cloud Console |

### Environment Variables

Set in n8n (Settings → Environment Variables):

```
IMGBB_API_KEY=your_imgbb_api_key
INSTAGRAM_BUSINESS_ACCOUNT_ID=your_ig_business_id
FACEBOOK_ACCESS_TOKEN=your_fb_access_token
FACEBOOK_PAGE_ID=your_fb_page_id
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_page_token
PINTEREST_BOARD_ID=your_pinterest_board_id
CAMPAIGN_LOG_SPREADSHEET_ID=your_google_sheet_id
SLACK_WEBHOOK_PATH=T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX
```

---

## Setup Instructions

### Step 1: Import the Workflow
1. Open your n8n instance
2. **Workflows** → **Import from File** → select `Social_Media_Campaign_Workflow.json`

### Step 2: Configure Credentials

#### OpenAI (HTTP Header Auth)
1. **Credentials** → **New** → **HTTP Header Auth**
2. Name: `Authorization` | Value: `Bearer sk-your-openai-key`
3. Used by both content generation and DALL-E poster nodes

#### Perplexity AI (separate HTTP Header Auth)
1. **Credentials** → **New** → **HTTP Header Auth**
2. Name: `Authorization` | Value: `Bearer pplx-your-api-key`

#### LinkedIn
1. Create LinkedIn App → request `w_member_social` scope
2. Add **LinkedIn OAuth2** credential in n8n

#### Facebook & Instagram
1. Create Meta App → add Facebook Login + Instagram Graph API
2. Generate long-lived Page Access Token
3. Get Instagram Business Account ID

#### Pinterest
1. Create Pinterest App → generate access token with `pins:write`
2. Create a board, note the board ID
3. Add as HTTP Header Auth: `Authorization: Bearer your-token`

#### Google Sheets
1. Enable Sheets API in Google Cloud Console
2. Add **Google Sheets OAuth2** credential
3. Create sheet with columns:
   **Date | Brand | Content Niche | Topic | Campaign Angle | Key Takeaway | Lead Hook / CTA | Platforms | Poster URL | Status | Leads Generated**

### Step 3: Assign Credentials

| Node | Credential |
|------|-----------|
| Generate Platform Content (OpenAI) | HTTP Header Auth (OpenAI) |
| Design Poster (DALL-E 3) | HTTP Header Auth (OpenAI) |
| AI Trend Analysis (Perplexity) | HTTP Header Auth (Perplexity) |
| Post to LinkedIn | LinkedIn OAuth2 |
| Post to Pinterest | HTTP Header Auth (Pinterest) |
| Log to Google Sheets | Google Sheets OAuth2 |

### Step 4: Activate
1. **Save** → Toggle **Active**
2. Or test with **Manual Trigger** first

---

## Lead Tracking

The Google Sheet automatically logs every campaign with a **"Leads Generated"** column (starts at 0). Update this manually or connect a form/webhook to track:

| Metric | Where to Track |
|--------|---------------|
| DMs received | Update "Leads Generated" column weekly |
| Page follows | Track follower growth per niche |
| Comments with intent | "Comment GROWTH" / "Interested" responses |
| Link clicks | Use UTM links in bio; track in Google Analytics |
| Pinterest traffic | Pinterest Analytics → Marvice board |

---

## Marvice Brand Guidelines in the Workflow

| Element | Value |
|---------|-------|
| Brand Name | **Marvice** (mentioned 1-2x per post) |
| Poster Colors | Deep blue, white, accent gold/orange |
| Poster Style | Clean corporate, modern sans-serif, professional |
| Voice | Confident, knowledgeable, approachable, results-driven |
| CTA per platform | LinkedIn: "DM us" / Instagram: "Follow + save" / Facebook: "Comment keyword" / Pinterest: SEO title |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| RSS feeds return empty | Google News RSS may rate-limit; add 2s delay |
| DALL-E rate limit | Add a Wait node (30s) before poster generation |
| Instagram 400 error | Verify Business Account ID and token permissions |
| LinkedIn 403 error | Reauthorize OAuth; check w_member_social scope |
| Pinterest 401 error | Regenerate access token; check pins:write scope |
| OpenAI 401 error | Check HTTP Header Auth: `Authorization: Bearer sk-xxx` |
| Posts don't mention Marvice | Check the system prompt in Generate Platform Content node |
