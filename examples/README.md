# Example: Rhino Concrete

## Live Dashboard

**URL:** https://rhino-call-analyzer.vercel.app

## Project Location

**Path:** `/root/clawd/projects/rhino-analyzer/`

## Stats

| Metric | Value |
|--------|-------|
| Total Calls | 415 |
| Date Range | Jan 2-Feb 3, 2026 |
| Customers | 49 (11.8%) |
| Spam | 71 (17.1%) |

| Operations | 9 (2.2%) |
| Incomplete | 211 (50.8%) |
| Transcribed | 244 |
| Avg Confidence | 95.2% |
| Missed Opportunities | 6 |

## Project Structure

```
rhino-analyzer/
├── data/
│   └── calls-merged.json       # Raw calls + transcripts merged
├── transcriptions/
│   └── {call_id}.json          # AssemblyAI results per call
├── analysis/
│   ├── classify.mjs            # Classification script
│   └── results-enhanced.json   # Final analysis
└── dashboard/
    ├── src/
    │   ├── App.jsx             # Main React component
    │   └── index.css           # Design tokens
    ├── public/
    │   └── data/
    │       └── results.json    # Deployed data
    └── dist/                   # Build output
```

## Key Files

### Classification Script
`analysis/classify.mjs` - Contains all classification logic and patterns

### Dashboard Components
`dashboard/src/App.jsx` - Contains:
- StatCard
- CategoryBadge
- SentimentBar
- AnswerBadge
- IncompleteReason
- AudioLink
- CallModal
- MissedOpportunitiesTab
- MethodologyTab

### Design System
`dashboard/src/index.css` - Dark mode tokens, mobile responsiveness

## Customization Points

For new customers, modify:
1. Company name in header
2. Patterns for industry-specific classification
3. Colors if needed (stick to design system)
4. Date range
