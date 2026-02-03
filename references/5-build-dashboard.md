# Step 5: Build Dashboard

## Use the interface-design Skill

**Skill location:** `/root/clawd/skills/interface-design/SKILL.md`

Follow its principles for:
- Consistent spacing (4px base)
- Color hierarchy
- Typography scale
- Dark mode tokens

## Tech Stack

- **React 18** - UI framework
- **Vite** - Build tool
- **Custom CSS** - No Tailwind/UI libraries

## Project Structure

```
dashboard/
├── index.html
├── package.json
├── vite.config.js
├── public/
│   └── data/
│       └── results.json    # Analysis results (loaded at runtime)
└── src/
    ├── main.jsx
    ├── App.jsx
    └── index.css           # Design tokens + components
```

## Design Tokens (Dark Mode)

```css
:root {
  /* Base Colors */
  --bg-base: #0a0a0b;
  --bg-surface-1: #111113;
  --bg-surface-2: #18181b;
  --bg-surface-3: #1f1f23;
  
  /* Text Hierarchy */
  --text-primary: #fafafa;
  --text-secondary: #a1a1aa;
  --text-muted: #71717a;
  --text-faint: #52525b;
  
  /* Borders */
  --border-default: rgba(255, 255, 255, 0.08);
  --border-subtle: rgba(255, 255, 255, 0.05);
  
  /* Category Colors */
  --color-customer: #22c55e;
  --color-spam: #ef4444;
  --color-operations: #3b82f6;
  --color-incomplete: #71717a;
  
  /* Spacing (4px base) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  
  /* Typography */
  --font-sans: 'Inter', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
  
  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 6px;
  --radius-lg: 8px;
}
```

## Dashboard Tabs

### 1. Call List (Main)
- Incomplete breakdown card
- Filter buttons (category + status)
- Date range filter
- Search input
- Calls table with click-to-expand

### 2. Missed Opportunities
- Alert card with count
- List of customer voicemails
- Each shows: location, phone, date, transcript preview, play button

### 3. Methodology
- Call breakdown tree (answered vs missed, with/without recording)
- Weekly breakdown table
- Duration vs Recording Duration explanation
- Why calls have no recording explanation

## Key Components

### StatCard
```jsx
function StatCard({ label, value, subtext, color }) {
  return (
    <div className="stat-card">
      <div className="stat-label">{label}</div>
      <div className="stat-value" style={{ color }}>{value}</div>
      {subtext && <div className="stat-subtext">{subtext}</div>}
    </div>
  );
}
```

### CategoryBadge
```jsx
function CategoryBadge({ category }) {
  const labels = {
    customer: 'Customer',
    spam: 'Spam',
    operations: 'Operations',
    incomplete: 'Incomplete'
  };
  return <span className={`badge badge-${category}`}>{labels[category]}</span>;
}
```

### SentimentBar
```jsx
function SentimentBar({ sentiment }) {
  if (!sentiment) return <span className="text-muted">—</span>;
  return (
    <div className="sentiment-bar">
      <div className="sentiment-positive" style={{ width: `${sentiment.positive}%` }} />
      <div className="sentiment-negative" style={{ width: `${sentiment.negative}%` }} />
      <div className="sentiment-neutral" style={{ width: `${sentiment.neutral}%` }} />
    </div>
  );
}
```

### CallModal
Full-screen modal showing:
- Recording link (opens in CallRail)
- All metadata (category, status, duration, date, phone, location)
- Sentiment breakdown
- Classification reasoning
- Full transcript with speaker labels

## Mobile Responsiveness

```css
@media (max-width: 768px) {
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  .stat-card {
    text-align: center;
  }
  .header {
    text-align: center;
  }
  .filters {
    justify-content: center;
  }
}
```

## Data Loading

Load from static JSON (no backend):

```jsx
useEffect(() => {
  fetch('./data/results.json')
    .then(r => r.json())
    .then(setData);
}, []);
```

## Filtering Logic

```jsx
let filtered = calls;

// Category filter
if (filter !== 'all') {
  filtered = filtered.filter(c => c.category === filter);
}

// Status filter
if (answerFilter === 'answered') {
  filtered = filtered.filter(c => c.answered === true);
}

// Date filter
if (dateFrom) {
  filtered = filtered.filter(c => new Date(c.start_time) >= new Date(dateFrom));
}

// Search
if (search) {
  const q = search.toLowerCase();
  filtered = filtered.filter(c => 
    c.customer_phone?.includes(q) ||
    c.transcript_full?.toLowerCase().includes(q)
  );
}
```
