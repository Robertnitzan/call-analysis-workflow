# Step 4: Classify Calls

## Categories

| Category | Color | Description |
|----------|-------|-------------|
| `customer` | 🟢 Green | Real customer inquiries, quotes, scheduling |
| `spam` | 🔴 Red | Robocalls, solicitation, scams |
| `b2b` | 🟠 Orange | Business calls, vendors, suppliers |
| `operations` | 🔵 Blue | Internal, employees, contractors |
| `incomplete` | ⚪ Gray | No recording, too short, unclear |

## Classification Logic

```javascript
function classifyCall(call) {
  const text = call.assemblyai_text || call.original_transcript || '';
  
  // 1. No transcript
  if (!text || text.trim().length < 20) {
    return { category: 'incomplete', reason: 'No transcript' };
  }
  
  // 2. Too short
  if (call.duration < 10 && text.split(' ').length < 15) {
    return { category: 'incomplete', reason: 'Too short' };
  }
  
  // 3. Check patterns in order
  if (matchesPatterns(text, SPAM_PATTERNS)) {
    return { category: 'spam', reason: 'Spam patterns detected' };
  }
  
  if (matchesPatterns(text, B2B_PATTERNS)) {
    return { category: 'b2b', reason: 'B2B/vendor patterns detected' };
  }
  
  if (matchesPatterns(text, OPERATIONS_PATTERNS)) {
    return { category: 'operations', reason: 'Operations patterns detected' };
  }
  
  if (matchesPatterns(text, CUSTOMER_PATTERNS)) {
    return { category: 'customer', reason: 'Customer patterns detected' };
  }
  
  // 4. Fallback
  if (call.direction === 'outbound') {
    return { category: 'operations', reason: 'Outbound call' };
  }
  
  if (call.speakers >= 2 && call.duration > 30) {
    return { category: 'customer', reason: 'Inbound conversation' };
  }
  
  return { category: 'incomplete', reason: 'Unable to classify' };
}
```

## Pattern Files

Patterns are stored in `patterns/` directory as JSON arrays of regex strings.

### patterns/spam-patterns.json
```json
[
  "warranty",
  "vehicle warranty",
  "medicare|medicaid",
  "health insurance",
  "credit card",
  "lower your rate",
  "solar panel",
  "press (one|1|two|2)",
  "business listing",
  "google listing",
  "final notice",
  "insurance agent",
  "refinance",
  "mortgage rate",
  "won a|winner",
  "subscription|renew",
  "donation|charity",
  "merchant service",
  "honors account|reward points",
  "not interested"
]
```

### patterns/b2b-patterns.json
```json
[
  "capital registration",
  "compensation insurance",
  "lumber company|lumber supply",
  "PRV engineers",
  "sperry griffin",
  "bidwell estimates",
  "managing principal",
  "reaching out for the owner",
  "detailed estimate service",
  "engineering service",
  "callrail"
]
```

### patterns/customer-patterns.json
```json
[
  "quote|estimate|bid",
  "how much|price|cost",
  "schedule|appointment",
  "project|job|work",
  "concrete|foundation|slab|driveway|patio",
  "remodel|renovation|repair",
  "bathroom|kitchen|basement",
  "interested in|looking for|need",
  "address is|located at|property",
  "call.*(back|me)|return.*call"
]
```

### patterns/operations-patterns.json
```json
[
  "delivery|delivering|dropped off",
  "schedule.*(pickup|delivery)",
  "material|supplies|order",
  "invoice|payment|bill",
  "employee|worker|crew",
  "contractor|subcontractor",
  "permit|inspection",
  "floor and decor|home depot",
  "truck|trailer|equipment",
  "job site"
]
```

## Confidence Scoring

Calculate confidence based on pattern matches:

```javascript
function calculateConfidence(text, patterns) {
  const matches = patterns.filter(p => new RegExp(p, 'i').test(text));
  // Base: 0.6, +0.1 per match, max 0.95
  return Math.min(0.6 + matches.length * 0.1, 0.95);
}
```

## Incomplete Reasons

Track WHY a call is incomplete:

| Reason | Description |
|--------|-------------|
| `no_recording` | Call has no recording (outbound, missed) |
| `too_short` | Duration < 10 seconds |
| `transcription_failed` | AssemblyAI couldn't transcribe |
| `unclear_content` | Has transcript but can't classify |

## Missed Opportunities

Special filter for customer voicemails:

```javascript
const missedOpportunities = calls.filter(c => 
  c.voicemail === true && 
  c.category === 'customer'
);
```

These are real leads that weren't answered and left a message.

## Output Format

```json
{
  "id": "CAL...",
  "category": "customer",
  "confidence_classification": 0.85,
  "reasoning": ["Customer patterns detected: 3 matches"],
  "key_topics": ["quote", "concrete", "schedule"],
  "incomplete_reason": null
}
```
