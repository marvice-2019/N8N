# Social Media Campaign Workflow - Setup Guide

## Workflow Overview

This n8n workflow automates a **multi-niche social media campaign** covering AI & Technology, Digital Marketing, Web Development, Corporate Gifting, and Marketing Updates.

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
         Perplexity AI (pick best niche topic)
                         ▼
         OpenAI GPT-4o (platform-specific content)
                         ▼
         DALL-E 3 (design HD poster)
                         ▼
         ImgBB (host image)
                         ▼
    ┌────────┬───────────┬──────────┐
    ▼        ▼           ▼          ▼
 LinkedIn  Instagram  Facebook  Pinterest
    └────────┴───────────┴──────────┘
                         ▼
         Google Sheets (log) → Slack (notify)
```

### Content Niches Covered

| Niche | Content Focus | Example Topics |
|-------|--------------|----------------|
| **AI & Technology** | Latest AI updates, tools, breakthroughs | GPT-5 launch, AI coding assistants, AI in healthcare |
| **Digital Marketing** | SEO, ads, social media strategy, analytics | Google algorithm update, TikTok ads ROI, email funnels |
| **Web Development** | Frontend, backend, frameworks, coding trends | React 20, Next.js updates, serverless architecture |
| **Corporate Gifting** | Business gifts, employee engagement, promo products | Holiday gift trends, branded merch, client appreciation |
| **Marketing Updates** | General marketing trends, branding, growth hacks | Influencer marketing, brand storytelling, viral campaigns |

### Workflow Stages

| Stage | Node(s) | Purpose |
|-------|---------|---------|
| 1. Trigger | Schedule (Mon-Fri 8AM) or Manual | Initiates the campaign |
| 2. Research | Google Trends + 4 Niche RSS Feeds | Fetches trends across 5 content niches |
| 3. Aggregate | Code Node | Merges all sources into unified topic list |
| 4. Analysis | Perplexity AI | Selects best niche topic & defines campaign angle |
| 5. Content | OpenAI GPT-4o (HTTP Request) | Generates niche-specific platform posts |
| 6. Design | DALL-E 3 (HTTP Request) | Creates HD niche-themed campaign poster |
| 7. Host | ImgBB | Hosts poster image with persistent URL |
| 8. Publish | LinkedIn, Instagram, Facebook, Pinterest | Posts in parallel |
| 9. Log | Google Sheets + Slack | Tracks campaigns with niche labels |

---

## Prerequisites & Credentials

### Required API Keys & Accounts

| Service | Purpose | How to Get |
|---------|---------|------------|
| **OpenAI API** | Content generation + DALL-E poster (via HTTP Request) | https://platform.openai.com/api-keys |
| **Perplexity API** | Trend analysis | https://docs.perplexity.ai/ |
| **ImgBB API** | Image hosting | https://api.imgbb.com/ |
| **LinkedIn OAuth2** | LinkedIn posting | LinkedIn Developer Portal → Create App |
| **Facebook/Instagram** | FB & IG posting | Meta Developer Portal → Graph API |
| **Pinterest API** | Pinterest pinning | Pinterest Developer Portal |
| **Google Sheets OAuth2** | Campaign logging | Google Cloud Console → Enable Sheets API |

### Environment Variables

Set these in your n8n instance (Settings → Environment Variables):

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
2. Go to **Workflows** → **Import from File**
3. Select `Social_Media_Campaign_Workflow.json`
4. Click **Import**

### Step 2: Configure Credentials

#### OpenAI (via HTTP Header Auth)
1. In n8n, go to **Credentials** → **New Credential**
2. Search for **HTTP Header Auth**
3. Set **Name**: `Authorization`
4. Set **Value**: `Bearer sk-your-openai-api-key`
5. Save — this single credential is used by both the content generation and DALL-E poster nodes

#### Perplexity AI
1. Create a **separate** HTTP Header Auth credential
2. Name: `Authorization`
3. Value: `Bearer pplx-your-api-key`

#### LinkedIn
1. Create a LinkedIn App at https://www.linkedin.com/developers/
2. Request `w_member_social` scope
3. In n8n, add **LinkedIn OAuth2** credential
4. Complete the OAuth flow

#### Facebook & Instagram
1. Create a Meta App at https://developers.facebook.com/
2. Add **Facebook Login** and **Instagram Graph API** products
3. Generate a long-lived Page Access Token
4. Get your Instagram Business Account ID from the API

#### Pinterest
1. Create a Pinterest App at https://developers.pinterest.com/
2. Generate an access token with `pins:write` scope
3. Create a board and note its ID
4. Add as HTTP Header Auth: `Authorization: Bearer your-token`

#### Google Sheets
1. Enable Google Sheets API in Google Cloud Console
2. Create OAuth2 credentials
3. In n8n, add **Google Sheets OAuth2** credential
4. Create a Google Sheet with columns: **Date, Content Niche, Topic, Campaign Angle, Key Takeaway, Platforms, Poster URL, Status**

### Step 3: Assign Credentials to Nodes
In the workflow, click each node and assign the correct credential:

| Node | Credential Type |
|------|----------------|
| `Generate Platform Content (OpenAI)` | HTTP Header Auth (OpenAI key) |
| `Design Poster (DALL-E 3)` | HTTP Header Auth (OpenAI key) |
| `AI Trend Analysis (Perplexity)` | HTTP Header Auth (Perplexity key) |
| `Post to LinkedIn` | LinkedIn OAuth2 |
| `Post to Pinterest` | HTTP Header Auth (Pinterest token) |
| `Log to Google Sheets` | Google Sheets OAuth2 |

**Important:** All AI nodes use standard **HTTP Request** nodes — no extra n8n packages or community nodes required. Works on any n8n version.

### Step 4: Adjust Schedule
The default schedule is **Mon-Fri at 8:00 AM** (weekdays). To change:
1. Click on **Campaign Schedule Trigger** node
2. Modify the cron expression:
   - Daily at 9 AM: `0 9 * * *`
   - Weekdays at 10 AM: `0 10 * * 1-5`
   - Twice a week (Tue/Thu): `0 8 * * 2,4`
   - Three times a week: `0 8 * * 1,3,5`

### Step 5: Activate
1. Click **Save** in the workflow editor
2. Toggle the workflow to **Active**
3. Or use **Manual Trigger** to test first

---

## How Content Niches Rotate

The workflow fetches trends from **all 5 niches simultaneously** each run, then Perplexity AI selects the **most engaging topic** from any niche. The AI is instructed to rotate between niches for variety, so your feed covers:

- **Monday**: AI & Technology update
- **Tuesday**: Digital Marketing tip
- **Wednesday**: Web Development trend
- **Thursday**: Corporate Gifting idea
- **Friday**: Marketing strategy update

*(Actual niche selection depends on what's trending — the AI picks the most viral-worthy topic)*

---

## Platform-Specific Notes

### LinkedIn
- Posts as a person (personal profile); change `postAs` to `organization` for company pages
- Thought-leadership tone with data points and actionable insights
- Image posts get 2x more engagement than text-only

### Instagram
- Requires a **Business** or **Creator** account connected to a Facebook Page
- Uses 2-step posting: Create Container → Publish
- Engaging captions with emojis and CTAs

### Facebook
- Posts to a **Page** (not personal profile)
- Conversational tone with engagement questions
- Page access token must have `pages_manage_posts` permission

### Pinterest
- Posts as Pins to a specific board
- SEO-rich descriptions with niche keywords for discoverability
- Great for Web Development infographics and Corporate Gifting ideas

---

## Customization Options

### Add More RSS Sources
Add additional HTTP Request nodes for more niche feeds:
- **Tech blogs**: TechCrunch, The Verge, Hacker News RSS
- **Marketing**: HubSpot Blog, Moz Blog, Search Engine Journal RSS
- **Web Dev**: CSS-Tricks, Smashing Magazine, Dev.to RSS
- **Gifting**: Corporate gifting industry newsletters

### Add More Platforms
Extend the workflow by adding nodes for:
- **Twitter/X** (via X API v2)
- **TikTok** (via TikTok Content Posting API)
- **YouTube Community** posts
- **Threads** (via Instagram API)

### Content Approval Gate
Insert a **Wait** node before posting to pause for human review:
1. Send content preview via email/Slack
2. Wait for approval webhook
3. Then proceed with posting

### A/B Testing
Duplicate content generation with different tones/angles and track performance in Google Sheets.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Google Trends returns empty | Check geo parameter; try different country codes |
| RSS feeds return empty | Google News RSS may be rate-limited; add a 2s delay between feeds |
| DALL-E rate limit | Add a Wait node (30s) before poster generation |
| Instagram 400 error | Verify Business Account ID and token permissions |
| LinkedIn 403 error | Reauthorize OAuth; check w_member_social scope |
| Pinterest 401 error | Regenerate access token; check pins:write scope |
| Image upload fails | Verify ImgBB API key; check image size < 32MB |
| OpenAI 401 error | Check HTTP Header Auth credential: `Authorization: Bearer sk-xxx` |
| Perplexity 401 error | Check HTTP Header Auth credential: `Authorization: Bearer pplx-xxx` |
