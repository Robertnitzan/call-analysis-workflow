# Step 1: Fetch Calls from CallRail

## Prerequisites

- CallRail API Key (read-only is sufficient)
- Account ID

## Get Credentials

1. Log in to CallRail
2. Go to Settings → API → API Keys
3. Create new key with read access
4. Copy the API key
5. Get Account ID from URL: `app.callrail.com/a/{ACCOUNT_ID}/...`

## API Request

```bash
curl -H "Authorization: Token token=YOUR_API_KEY" \
  "https://api.callrail.com/v3/a/ACCOUNT_ID/calls.json?per_page=250&page=1"
```

### Pagination

CallRail returns max 250 calls per page. Loop through pages:

```javascript
async function fetchAllCalls(apiKey, accountId) {
  const calls = [];
  let page = 1;
  
  while (true) {
    const url = `https://api.callrail.com/v3/a/${accountId}/calls.json?per_page=250&page=${page}`;
    const res = await fetch(url, {
      headers: { 'Authorization': `Token token=${apiKey}` }
    });
    const data = await res.json();
    
    if (!data.calls || data.calls.length === 0) break;
    calls.push(...data.calls);
    page++;
  }
  
  return calls;
}
```

### Date Filtering

```bash
# Calls from January 2026
?start_date=2026-01-01&end_date=2026-01-31
```

### Company Filtering

```bash
# Specific company only
?company_id=COM123456
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Call ID (CAL...) |
| `direction` | string | "inbound" or "outbound" |
| `duration` | number | Total call duration in seconds |
| `recording_duration` | number | Recording length in seconds |
| `start_time` | string | ISO timestamp |
| `answered` | boolean | Was the call answered |
| `voicemail` | boolean | Did caller leave voicemail |
| `customer_phone_number` | string | Caller's phone |
| `customer_city` | string | Caller's location |
| `customer_state` | string | Caller's state |
| `recording` | string | URL to recording API endpoint |
| `recording_player` | string | URL to web player (with access_key) |
| `transcription` | string | CallRail's basic transcript |
| `company_id` | string | Company ID |
| `company_name` | string | Company name |
| `source` | string | Call source/campaign |

## Save Calls

Save each call to a separate JSON file for processing:

```javascript
const fs = require('fs');

calls.forEach(call => {
  fs.writeFileSync(
    `calls/${call.id}.json`,
    JSON.stringify(call, null, 2)
  );
});
```

## Output

```
data/
└── calls/
    ├── CAL019b8007176f785eb3919ead3cb7ee6d.json
    ├── CAL019b84d9b278790d904c5021cdd11263.json
    └── ...
```
