# Social Media Campaign Workflow - Setup Guide

## Workflow Overview

This n8n workflow automates your entire social media campaign pipeline:

```
Schedule Trigger → Fetch Trends → AI Analysis → Content Generation → Poster Design → Auto-Post
```

### Workflow Stages

| Stage | Node(s) | Purpose |
|-------|---------|---------|
| 1. Trigger | Schedule (Mon/Wed/Fri 8AM) or Manual | Initiates the campaign |
| 2. Research | Google Trends API | Fetches daily trending topics |
| 3. Analysis | Perplexity AI | Selects best topic & defines campaign angle |
| 4. Content | OpenAI GPT-4o | Generates platform-specific posts |
| 5. Design | DALL-E 3 | Creates HD campaign poster |
| 6. Host | ImgBB | Hosts poster image with persistent URL |
| 7. Publish | LinkedIn, Instagram, Facebook, Pinterest | Posts in parallel |
| 8. Log | Google Sheets + Slack | Tracks campaigns & notifies team |

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
1. Create an **HTTP Header Auth** credential
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
4. Create a Google Sheet with columns: Date, Topic, Campaign Angle, Platforms, Poster URL, Status

### Step 3: Update Credential References
In the workflow, update the credential IDs in these nodes:
- `Generate Platform Content (OpenAI)` → HTTP Header Auth credential (Authorization: Bearer sk-xxx)
- `Design Poster (DALL-E 3)` → Same HTTP Header Auth credential as above
- `AI Trend Analysis (Perplexity)` → HTTP Header Auth credential (Authorization: Bearer pplx-xxx)
- `Post to LinkedIn` → LinkedIn OAuth2 credential
- `Post to Pinterest` → HTTP Header Auth credential (Authorization: Bearer pinterest-token)
- `Log to Google Sheets` → Google Sheets OAuth2 credential

**Note:** All AI nodes (OpenAI GPT-4o, DALL-E 3, Perplexity) use standard **HTTP Request** nodes instead of custom OpenAI nodes. This ensures compatibility with all n8n versions without needing to install extra community nodes.

### Step 4: Adjust Schedule
The default schedule is Mon/Wed/Fri at 8:00 AM. To change:
1. Click on **Campaign Schedule Trigger** node
2. Modify the cron expression:
   - Daily at 9 AM: `0 9 * * *`
   - Weekdays at 10 AM: `0 10 * * 1-5`
   - Twice a week (Tue/Thu): `0 8 * * 2,4`

### Step 5: Activate
1. Click **Save** in the workflow editor
2. Toggle the workflow to **Active**
3. Or use **Manual Trigger** to test first

---

## Platform-Specific Notes

### LinkedIn
- Posts as a person (personal profile); change `postAs` to `organization` for company pages
- Image posts get 2x more engagement than text-only

### Instagram
- Requires a **Business** or **Creator** account connected to a Facebook Page
- Uses 2-step posting: Create Container → Publish
- Supports single image posts; extend for carousels by modifying the container creation

### Facebook
- Posts to a **Page** (not personal profile)
- Uses the Photos API endpoint for image + text posts
- Page access token must have `pages_manage_posts` permission

### Pinterest
- Posts as Pins to a specific board
- SEO-rich descriptions improve discoverability
- Supports image_url source type for direct URL posting

---

## Customization Options

### Change Trending Topics Source
Replace the Google Trends node with:
- **Twitter/X Trending API** for real-time social trends
- **Reddit API** for community-driven topics
- **NewsAPI** for news-based campaigns
- **RSS Feed** for industry-specific content

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
| DALL-E rate limit | Add a Wait node (30s) before poster generation |
| Instagram 400 error | Verify Business Account ID and token permissions |
| LinkedIn 403 error | Reauthorize OAuth; check w_member_social scope |
| Pinterest 401 error | Regenerate access token; check pins:write scope |
| Image upload fails | Verify ImgBB API key; check image size < 32MB |
