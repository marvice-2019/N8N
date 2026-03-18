# Marvice & Conxyou - Lead Gen, Blog & Engagement Campaign

## Purpose

This n8n workflow runs **two brands** automatically:
- **Marvice** — AI & Technology, Digital Marketing, Web Development
- **Conxyou** — Corporate Gifting, business gifts, employee engagement

### What It Does
- Auto-posts engaging content to **LinkedIn, Instagram, Facebook, Pinterest**
- Auto-publishes a **full SEO blog post to WordPress** for every campaign
- Sends **email notification** when campaign is live
- Logs everything to **Google Sheets** for lead tracking
- Smart branding: Marvice mentioned only sometimes (not every post), Conxyou always on gifting content
- Every post includes a **"Comment to get the full blog"** CTA in the caption (NOT on the poster)

---

## Brand Rules

| Niche | Brand | Mention Rule |
|-------|-------|-------------|
| **AI & Technology** | Marvice | Sometimes — varies between pure value posts and branded posts |
| **Digital Marketing** | Marvice | Sometimes — not every time, keeps it natural |
| **Web Development** | Marvice | Always mentioned |
| **Corporate Gifting** | Conxyou | Always mentioned |
| **Marketing Updates** | Marvice | Sometimes |

The AI decides per-campaign whether to include the brand name (via `mention_brand: true/false`). When `false`, the post is pure value content with no brand mention — just insights and tips.

---

## Workflow Pipeline

```
Schedule (Mon-Fri 8AM) or Manual Trigger
         ▼
    ┌─── Google Trends ───────────────────────┐
    ├─── AI & Tech News (RSS) ────────────────┤
    ├─── Digital Marketing News (RSS) ────────┤── Aggregate Trends
    ├─── Web Development News (RSS) ──────────┤
    └─── Corporate Gifting News (RSS) ────────┘
         ▼
    Perplexity AI → Pick topic + assign brand (Marvice or Conxyou)
         ▼
    OpenAI GPT-4o → Platform posts + blog_content (800-1200 words)
         ▼
    DALL-E 3 → Clean poster (NO text, NO CTA on image)
         ▼
    ImgBB → Host image
         ▼
    ┌────────┬───────────┬──────────┐
    ▼        ▼           ▼          ▼
 LinkedIn  Instagram  Facebook  Pinterest
    └────────┴───────────┴──────────┘
         ▼
    Campaign Summary
         ▼
    ┌────────────────┬──────────────────┬──────────────────┐
    ▼                ▼                  ▼
 Google Sheets    WordPress Blog    Email Notification
 (lead tracking)  (auto-publish)    (campaign summary)
```

---

## Post CTAs (in captions, NOT on poster)

Every social media caption includes engagement-driving CTAs:

| Platform | Comment/Blog CTA |
|----------|-----------------|
| **LinkedIn** | "Comment below to get the detailed blog on this topic" |
| **Instagram** | "Drop a comment and we'll send you the full breakdown" / "Save this" |
| **Facebook** | "Comment for the full blog" / engagement question |
| **Pinterest** | SEO description drives traffic to blog |

The **poster image is always clean** — no text, no CTA overlaid on it.

---

## New Features

### Auto-Blog to WordPress
- Generates 800-1200 word SEO blog post with H2/H3 headings
- Auto-publishes to your WordPress site via REST API
- Blog excerpt = key takeaway from the campaign

### Email Notification (replaces Slack)
- Sends HTML email when campaign is published
- Includes: brand, topic, angle, lead hook, blog title, poster link
- Uses SendGrid API (or swap for SMTP)

---

## Prerequisites & Credentials

### Required API Keys

| Service | Purpose | How to Get |
|---------|---------|------------|
| **OpenAI API** | Content + DALL-E poster + blog | https://platform.openai.com/api-keys |
| **Perplexity API** | Trend analysis | https://docs.perplexity.ai/ |
| **ImgBB API** | Image hosting | https://api.imgbb.com/ |
| **LinkedIn OAuth2** | LinkedIn posting | LinkedIn Developer Portal |
| **Facebook/Instagram** | FB & IG posting | Meta Developer Portal |
| **Pinterest API** | Pinterest pinning | Pinterest Developer Portal |
| **Google Sheets OAuth2** | Lead tracking | Google Cloud Console |
| **WordPress** | Blog publishing | WordPress Admin → Users → Application Passwords |
| **SendGrid API** | Email notification | https://sendgrid.com/ (or use SMTP) |

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
WORDPRESS_URL=https://yourdomain.com
NOTIFICATION_EMAIL=your@email.com
FROM_EMAIL=noreply@yourdomain.com
```

---

## Setup Instructions

### Step 1: Import the Workflow
1. Open your n8n instance
2. **Workflows** → **Import from File** → select `Social_Media_Campaign_Workflow.json`

### Step 2: Configure Credentials

#### OpenAI (HTTP Header Auth)
- Name: `Authorization` | Value: `Bearer sk-your-openai-key`

#### Perplexity AI (separate HTTP Header Auth)
- Name: `Authorization` | Value: `Bearer pplx-your-api-key`

#### WordPress (HTTP Header Auth)
1. WordPress Admin → Users → Application Passwords → Generate
2. Create HTTP Header Auth: Name: `Authorization` | Value: `Basic base64(username:app_password)`
3. To get base64: `echo -n "username:app_password" | base64`

#### SendGrid (HTTP Header Auth)
- Name: `Authorization` | Value: `Bearer SG.your-sendgrid-api-key`
- **Alternative**: Replace the SendGrid node with n8n's built-in "Send Email" node using SMTP

#### LinkedIn, Facebook, Instagram, Pinterest, Google Sheets
- Same as before (see credential setup in each node's notes)

### Step 3: Google Sheet Columns

Create a sheet named "Campaign Log" with columns:

**Date | Brand | Content Niche | Topic | Campaign Angle | Key Takeaway | Lead Hook / CTA | Blog Title | Platforms | Poster URL | Blog Published | Status | Leads Generated**

### Step 4: Activate
1. **Save** → Toggle **Active**
2. Or test with **Manual Trigger** first

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Wrong brand on gifting posts | Parse AI Analysis enforces Conxyou for Corporate Gifting |
| Marvice mentioned every time | Check `mention_brand` field — should be false sometimes for AI/Marketing |
| CTA showing on poster | Poster prompt says "NO text on image" — check DALL-E prompt |
| WordPress 401 error | Verify Application Password and base64 encoding |
| Email not sending | Check SendGrid API key and FROM_EMAIL (must be verified sender) |
| Blog content missing | Check Parse Generated Content node — blog_content field |
