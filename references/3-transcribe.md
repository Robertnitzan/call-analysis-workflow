# Step 3: Transcribe with AssemblyAI

## Use the assemblyai-transcribe Skill

**Skill location:** `/root/clawd/skills/assemblyai-transcribe/SKILL.md`

**Required:** `ASSEMBLYAI_API_KEY` environment variable

## Transcription Options

```json
{
  "speaker_labels": true,
  "sentiment_analysis": true,
  "language_detection": true
}
```

| Option | Purpose |
|--------|---------|
| `speaker_labels` | Identify who said what (Speaker A, B) |
| `sentiment_analysis` | Detect positive/negative/neutral per sentence |
| `language_detection` | Auto-detect language |

## Upload and Transcribe Flow

```javascript
async function transcribeCall(audioPath, apiKey) {
  // 1. Upload audio
  const audioData = fs.readFileSync(audioPath);
  const uploadRes = await fetch('https://api.assemblyai.com/v2/upload', {
    method: 'POST',
    headers: {
      'Authorization': apiKey,
      'Content-Type': 'application/octet-stream'
    },
    body: audioData
  });
  const { upload_url } = await uploadRes.json();
  
  // 2. Submit transcription job
  const jobRes = await fetch('https://api.assemblyai.com/v2/transcript', {
    method: 'POST',
    headers: {
      'Authorization': apiKey,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      audio_url: upload_url,
      speaker_labels: true,
      sentiment_analysis: true,
      language_detection: true
    })
  });
  const job = await jobRes.json();
  
  // 3. Poll for completion
  while (true) {
    await new Promise(r => setTimeout(r, 3000));
    
    const pollRes = await fetch(`https://api.assemblyai.com/v2/transcript/${job.id}`, {
      headers: { 'Authorization': apiKey }
    });
    const result = await pollRes.json();
    
    if (result.status === 'completed') return result;
    if (result.status === 'error') throw new Error(result.error);
  }
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `text` | string | Full transcript |
| `confidence` | number | 0-1, transcription accuracy |
| `audio_duration` | number | Duration in seconds |
| `language_code` | string | Detected language (e.g., "en") |
| `utterances` | array | Speaker-labeled segments |
| `sentiment_analysis_results` | array | Sentiment per sentence |

### Utterances Format

```json
{
  "speaker": "A",
  "text": "Hello, how can I help you?",
  "start": 0,
  "end": 2500,
  "confidence": 0.95
}
```

### Sentiment Format

```json
{
  "text": "That's great, thank you!",
  "sentiment": "POSITIVE",
  "confidence": 0.92,
  "start": 5000,
  "end": 7000
}
```

## Calculate Sentiment Summary

```javascript
function getSentimentSummary(sentimentResults) {
  if (!sentimentResults?.length) return null;
  
  const counts = { POSITIVE: 0, NEGATIVE: 0, NEUTRAL: 0 };
  sentimentResults.forEach(s => counts[s.sentiment]++);
  
  const total = sentimentResults.length;
  return {
    positive: Math.round(counts.POSITIVE / total * 100),
    negative: Math.round(counts.NEGATIVE / total * 100),
    neutral: Math.round(counts.NEUTRAL / total * 100),
    total_segments: total
  };
}
```

## Batch Processing

```javascript
async function transcribeAllCalls(calls, apiKey) {
  const results = {};
  
  for (const call of calls) {
    const audioPath = `./audio/${call.id}.mp3`;
    if (!fs.existsSync(audioPath)) continue;
    
    console.log(`Transcribing ${call.id}...`);
    const result = await transcribeCall(audioPath, apiKey);
    
    results[call.id] = {
      call_id: call.id,
      assemblyai: {
        transcript_id: result.id,
        text: result.text,
        confidence: result.confidence,
        language: result.language_code,
        utterances: result.utterances,
        sentiment_analysis_results: result.sentiment_analysis_results
      }
    };
    
    // Save progress
    fs.writeFileSync(
      `./transcriptions/${call.id}.json`,
      JSON.stringify(results[call.id], null, 2)
    );
  }
  
  return results;
}
```

## Expected Results

- **Confidence:** 90-99% for clear calls, 70-85% for noisy/accented
- **Processing time:** ~30 seconds per minute of audio
- **Speakers:** Usually 2 (customer + business)

## Costs

AssemblyAI pricing: ~$0.00025 per second of audio
- 1 minute call = ~$0.015
- 100 calls × 2 min avg = ~$3

Free tier: 100 hours/month
