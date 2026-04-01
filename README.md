# News Alert Agent - Insurance Content Platform

> AI-powered daily insurance content platform for Chinese-American insurance brokers, built on n8n automation workflows.

## Project Overview

This platform automatically generates and serves Chinese-language insurance content tailored for US-market insurance brokers to share with their Chinese-American clients. It combines news aggregation, AI-powered translation/rewriting, and a web-based content portal.

### Core Relationship

```
Insurance Broker (platform user) → selects content → forwards to → US Chinese-American Clients (reader)
```

### Key Capabilities

- Daily top-10 insurance news aggregation from multiple sources
- AI-powered English-to-Chinese translation and client-friendly rewriting
- Automated content library generation (education, case studies, product intros)
- AI-generated article images via Pollinations
- Web portal with filtering, archiving, and card-based layout
- Scheduled daily execution with no manual intervention

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Content Production Layer (n8n)             │
│                                                              │
│   Workflow A (7:00 AM)          Workflow B (8:00 AM)         │
│   Daily Newsletter              Content Library Generator    │
│   3 RSS Sources → Dedup →       Weekly Topic Rotation →      │
│   Score → Top 10 →              AI Generate → Format →       │
│   AI Translate/Rewrite →        Store                        │
│   Format → Store                                             │
│                                                              │
│   ┌──────────────────────────────────────────────────────┐   │
│   │           Storage Layer (Google Sheets)                │   │
│   │   Sheet: newsletter (15 fields)                       │   │
│   │   Sheet: content_library (13 fields)                  │   │
│   └──────────────────────┬───────────────────────────────┘   │
│                          │                                    │
│   ┌──────────────────────▼───────────────────────────────┐   │
│   │      Workflow C - Web Display Layer (Webhook)         │   │
│   │      Read Sheets → Generate HTML → Serve Page         │   │
│   │      Supports: Home / Newsletter / Library / Archive  │   │
│   └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## Workflows

### Workflow A: Daily Insurance Newsletter

**File:** `workflows/workflow-a-daily-newsletter.json`

| Node | Function |
|------|----------|
| Daily 7AM Trigger | Cron schedule, daily at 7:00 AM |
| RSS - Insurance General | Google News RSS: general US insurance |
| RSS - Life Insurance | Google News RSS: life insurance |
| RSS - Annuity Retirement | Google News RSS: annuity & retirement |
| Merge Feeds 1 & 2 | Combine all 3 RSS sources |
| Dedup Score Filter | Remove duplicates, keyword relevance scoring, filter low-quality |
| Limit Top 10 | Keep highest-scoring 10 articles |
| AI Translate Rewrite | Gemini 1.5 Flash: translate, rewrite, classify, generate image prompt |
| Parse & Build Output | Parse AI JSON response, build image URLs, handle errors |
| Save to Newsletter Sheet | Append 15-field record to Google Sheets |

**Scoring Logic:**
- High-value keywords (+15): `life insurance`, `annuity`, `retirement`, `death benefit`, `whole life`, `term life`, etc.
- Mid-value keywords (+5): `insurance`, `coverage`, `premium`, `policy`, `beneficiary`, etc.
- Negative keywords (-20): `auto insurance`, `car insurance`, `home insurance`, `pet insurance`, etc.
- Threshold: items scoring < 5 are discarded

### Workflow B: Content Library Generator

**File:** `workflows/workflow-b-content-library.json`

| Node | Function |
|------|----------|
| Daily 8AM Trigger | Cron schedule, daily at 8:00 AM |
| Generate Topic List | Rotating weekly topics (14 topics across 7 days) |
| Split In Batches | Process topics one at a time |
| AI Generate Content | Gemini 1.5 Flash: generate 400-600 word articles |
| Parse Library Content | Parse AI output, build image URLs |
| Save to Library Sheet | Append 13-field record to Google Sheets |

**Weekly Topic Rotation:**

| Day | Topic 1 | Topic 2 |
|-----|---------|---------|
| Sun | Family case study | Term vs Whole Life education |
| Mon | Annuity product types | Retirement planning 101 |
| Tue | Icebreaker article | Dual-goal planning case |
| Wed | Tax advantages of life insurance | IUL explainer |
| Thu | Family risk checklist | Young client outreach |
| Fri | Market trends | Retirement income case study |
| Sat | Cultural perspectives | Retirement anxiety solutions |

**Content Types:** `education`, `product_intro`, `case_study`, `client_communication`

**Categories:** `life_insurance`, `annuity`, `retirement`, `family_finance`, `industry_news`, `regulation`

### Workflow C: Web Platform

**File:** `workflows/workflow-c-web-platform.json`

| Node | Function |
|------|----------|
| Web Request | Webhook endpoint at `/webhook/insurance-portal` |
| Parse Route | Extract query params: page, date, category |
| Read Newsletter Data | Fetch all newsletter records from Sheets |
| Read Library Data | Fetch all library records from Sheets |
| Merge Data | Combine both datasets |
| Generate HTML Page | Build full responsive HTML with CSS |
| Respond HTML | Return HTML with Content-Type header |

**Web Features:**
- Responsive card-grid layout with article images
- Date and category dropdown filters
- Today's newsletter section
- Content library section
- Historical archive navigation (last 30 days)
- Article detail view with full HTML content
- Mobile-friendly design
- 5-minute cache headers

**URL Examples:**
```
/webhook/insurance-portal                          → Homepage (today's content)
/webhook/insurance-portal?date=2025-01-15          → Specific date
/webhook/insurance-portal?category=life_insurance   → Filter by category
/webhook/insurance-portal?page=article&id=3         → Article detail
```

---

## Data Schema

### Newsletter Sheet (15 fields)

| Field | Description |
|-------|-------------|
| Date | Fetch date (YYYY-MM-DD) |
| Category | life_insurance / annuity / retirement / family_finance / industry_news / regulation |
| Chinese Title | AI-generated Chinese headline |
| Chinese Summary | One-line Chinese summary (< 50 chars) |
| Chinese Content HTML | Full article with HTML markup |
| Tags | Comma-separated topic tags |
| Image URL | Pollinations AI generated image URL |
| Image Prompt | English image description for AI generation |
| Original Title | Source English headline |
| Original Link | Source article URL |
| Original Pub Date | Source publication timestamp |
| Publish Timestamp | ISO timestamp when processed |
| Status | `pending_review` / `published` / `rejected` |
| Content Type | `newsletter` |
| Relevance Score | Keyword-based relevance score |

### Content Library Sheet (13 fields)

| Field | Description |
|-------|-------------|
| Date | Generation date (YYYY-MM-DD) |
| Category | Content category |
| Content Type | education / product_intro / case_study / client_communication |
| Chinese Title | AI-generated Chinese headline |
| Chinese Summary | One-line Chinese summary |
| Chinese Content HTML | Full article with HTML markup |
| Tags | Comma-separated topic tags |
| Target Audience | Intended reader segment |
| Image URL | Pollinations AI generated image URL |
| Image Prompt | English image description |
| Publish Timestamp | ISO timestamp |
| Status | `pending_review` / `published` / `rejected` |
| Source | `ai_generated` |

---

## Setup Guide

### Prerequisites

- n8n instance (v1.0+, self-hosted or cloud)
- Google Gemini API key
- Google Sheets OAuth2 credentials
- Google Sheets document with two sheet tabs: `newsletter` and `content_library`

### Step-by-Step

1. **Create Google Sheet**
   - Create a new Google Sheets document
   - Add sheet tab `newsletter` with 15 column headers (see schema above)
   - Add sheet tab `content_library` with 13 column headers (see schema above)

2. **Configure n8n Credentials**
   - Add credential `googleGeminiApi` with your Gemini API key
   - Add credential `googleSheetsOAuth2Api` with Google OAuth2 setup

3. **Import Workflows**
   - Import `workflow-a-daily-newsletter.json` into n8n
   - Import `workflow-b-content-library.json` into n8n
   - Import `workflow-c-web-platform.json` into n8n

4. **Configure Workflow Parameters**
   - In each workflow, update the Google Sheets `documentId` with your Sheet ID
   - Verify sheet names match: `newsletter` and `content_library`

5. **Activate**
   - Activate Workflow A and B (they run on daily schedules)
   - Activate Workflow C (webhook stays active for web access)
   - Access the web portal at: `https://<your-n8n-domain>/webhook/insurance-portal`

---

## Content Strategy

### Image Generation

- **Source:** Pollinations AI (free, no API key required)
- **Method:** AI generates English image descriptions per article → URL-encoded into Pollinations endpoint
- **Style:** Warm, professional, high-quality visuals
- **Fallback:** Frontend hides broken images gracefully via `onerror`

### Quality Control

- Keyword-based relevance scoring with negative keyword filtering
- Deduplication via normalized title comparison
- AI JSON parsing with try-catch fallback
- All content defaults to `pending_review` status (supports manual review workflow)
- Non-target insurance types (auto, home, pet, travel) automatically excluded

### Content Tone

- Professional yet approachable
- Tailored for Chinese-American families
- Explains English terms with Chinese context on first mention
- Naturally guides readers toward financial awareness (no hard selling)
- Covers: icebreaking, education, trust building, product understanding, conversion support

---

## Future Roadmap

- [ ] Multi-language output (English version)
- [ ] Supabase/database backend (replace Google Sheets for scale)
- [ ] Email/WeChat push distribution
- [ ] Broker-personalized content templates
- [ ] Client segment-based recommendations
- [ ] Manual review dashboard
- [ ] Search and favorites functionality
- [ ] Multi-channel publishing (social media, SMS)

---

## License

Private project. All rights reserved.

## Workflow Review

See `WORKFLOW_REVIEW_AND_UPGRADE_PLAN.md` for a production-grade n8n upgrade plan and a detailed audit of the current workflows.
