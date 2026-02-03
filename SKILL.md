---
name: call-analysis-workflow
description: Build call analysis dashboards from CallRail data. Downloads calls, transcribes with AssemblyAI, classifies by category, and generates a professional React dashboard.
---

# Call Analysis Workflow

Build professional call analysis dashboards for businesses using CallRail.

## Overview

This workflow transforms raw CallRail data into an insightful dashboard showing:
- Call categories (Customer, Spam, Operations, Incomplete)
- Missed opportunities (customer voicemails that need follow-up)
- Sentiment analysis per call
- Transcripts with speaker diarization
- Weekly breakdowns and methodology explanations

## Required Skills

This workflow uses two existing skills:

### 1. assemblyai-transcribe
**Location:** `/root/clawd/skills/assemblyai-transcribe/SKILL.md`
**Purpose:** Transcribe call recordings with speaker labels and sentiment analysis
**Required:** `ASSEMBLYAI_API_KEY` in environment

### 2. interface-design
**Location:** `/root/clawd/skills/interface-design/SKILL.md`
**Purpose:** Design consistent, professional dashboard UI
**Key principles:** Dark mode, borders-only depth, 4px spacing, monospace for data

## Workflow Steps

### Step 1: Fetch Calls from CallRail

See: `references/1-fetch-calls.md`

**Required credentials:**
- CallRail API Key (Settings → API → API Keys)
- Account ID (from CallRail URL)

**API endpoint:**
```
GET https://api.callrail.com/v3/a/{account_id}/calls.json
Authorization: Token token={api_key}
```

**Key fields to extract:**
- `id`, `direction`, `duration`, `start_time`
- `customer_phone_number`, `customer_city`, `customer_state`
- `answered`, `voicemail`
- `recording` (URL to get audio)
- `transcription` (CallRail's basic transcript)

### Step 2: Download Recordings

See: `references/2-download-recordings.md`

For each call with a recording:
1. GET `{recording_url}.json` with auth header
2. Follow redirect URL to download MP3
3. Save to `audio/{call_id}.mp3`

**Note:** Only calls with `recording` field have audio. Outbound calls and missed calls without voicemail have no recording.

### Step 3: Transcribe with AssemblyAI

See: `references/3-transcribe.md`

Use the `assemblyai-transcribe` skill with these options:
```json
{
  "speaker_labels": true,
  "sentiment_analysis": true,
  "language_detection": true
}
```

**Output per call:**
- `text` - Full transcript
- `confidence` - Transcription accuracy (0-1)
- `utterances` - Array of {speaker, text, start, end}
- `sentiment_analysis_results` - Array of {text, sentiment, confidence}

### Step 4: Classify Calls

See: `references/4-classify.md`

**Categories:**
| Category | Description | Patterns |
|----------|-------------|----------|
| customer | Real customer inquiries | quote, estimate, price, schedule, project |
| spam | Robocalls, solicitation | warranty, medicare, press 1, insurance |

| operations | Internal/employee calls | delivery, materials, job site |
| incomplete | Can't classify | No recording, too short, unclear |

**Classification logic:**
1. No transcript → `incomplete`
2. Check spam patterns → `spam` if matched

4. Check customer patterns → `customer` if matched
5. Check operations patterns → `operations` if matched
6. Fallback by direction: outbound → `operations`, inbound → `customer` or `incomplete`

See `patterns/` directory for full pattern lists.

### Step 5: Build Dashboard

See: `references/5-build-dashboard.md`

**Tech stack:**
- React 18 + Vite
- Tailwind-style CSS (custom tokens)
- No external UI libraries

**Dashboard structure:**
```
├── Tabs: Call List | Missed Opportunities | Methodology
├── Stats Grid: Total, Customers, Spam, Operations, Answered, Missed
├── Filters: Category, Status, Date Range, Search
├── Call Table: Date, Status, Duration, Category, Confidence, Sentiment, Recording
└── Call Detail Modal: Full transcript, speaker labels, sentiment breakdown
```

**Design tokens (dark mode):**
```css
--bg-base: #0a0a0b
--bg-surface-1: #111113
--text-primary: #fafafa
--text-muted: #71717a
--color-customer: #22c55e
--color-spam: #ef4444

```

### Step 6: Deploy to Vercel

See: `references/6-deploy.md`

```bash
npm run build
cp -r public/data dist/
vercel --prod --yes
```

## Data Structure

### calls-merged.json
```json
{
  "id": "CAL...",
  "direction": "inbound|outbound",
  "duration": 120,
  "recording_duration": 110,
  "start_time": "2026-01-15T10:30:00-08:00",
  "customer_phone": "+1234567890",
  "customer_city": "San Francisco",
  "answered": true,
  "voicemail": false,
  "has_recording": true,
  "has_assemblyai": true,
  "confidence": 0.95,
  "assemblyai_text": "Full transcript...",
  "utterances": [...],
  "sentiment_results": [...]
}
```

### results.json
```json
{
  "generated_at": "2026-02-03T...",
  "stats": {
    "total": 415,
    "by_category": { "customer": 63, "spam": 129, "operations": 12, ... },
    "answered": 309,
    "unanswered": 106,
    "voicemail": 38
  },
  "calls": [
    {
      "...merged fields...",
      "category": "customer",
      "confidence_classification": 0.85,
      "reasoning": ["Customer patterns detected"],
      "sentiment_summary": { "positive": 40, "negative": 10, "neutral": 50 },
      "incomplete_reason": null
    }
  ]
}
```

## Key Insights

### Why calls have no recording
- **Outbound calls:** CallRail doesn't record by default
- **Missed without voicemail:** Nothing to record
- **Softphone calls:** May not route through tracking number

### Duration vs Recording Duration
- **Duration:** Total call time (includes ringing)
- **Recording Duration:** Only recorded portion (starts after answer)
- **Average difference:** ~9 seconds (ringing time)

### Missed Opportunities
Customer voicemails = leads that weren't answered. Filter by:
```javascript
calls.filter(c => c.voicemail && c.category === 'customer')
```

## Example Project

See: `examples/rhino-concrete/`

**Live dashboard:** https://rhino-call-analyzer.vercel.app

**Stats:**
- 415 total calls (Jan 2026)
- 63 customers, 129 spam, 12 operations, 211 incomplete
- 244 calls transcribed with AssemblyAI (95.2% avg confidence)
- 6 missed opportunities identified
