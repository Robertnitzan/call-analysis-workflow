# Step 2: Download Recordings

## Which Calls Have Recordings

Not all calls have recordings:
- ✅ Answered inbound calls
- ✅ Voicemails (the message portion)
- ❌ Outbound calls (usually not recorded)
- ❌ Missed calls without voicemail
- ❌ Softphone calls (may not route through tracking)

Check: `call.recording !== null`

## Get Recording URL

The `recording` field is an API endpoint, not a direct audio URL.

```javascript
async function getRecordingUrl(call, apiKey) {
  const res = await fetch(`${call.recording}.json`, {
    headers: { 'Authorization': `Token token=${apiKey}` }
  });
  const data = await res.json();
  return data.url; // Redirect URL to actual audio
}
```

## Download Audio

The redirect URL points to a signed S3 URL. Download it:

```javascript
async function downloadRecording(call, apiKey, outputDir) {
  // Get redirect URL
  const apiUrl = `https://api.callrail.com/v3/a/${accountId}/calls/${call.id}/recording.json`;
  const res = await fetch(apiUrl, {
    headers: { 'Authorization': `Token token=${apiKey}` }
  });
  const { url } = await res.json();
  
  // Download audio
  const audioRes = await fetch(url);
  const buffer = Buffer.from(await audioRes.arrayBuffer());
  
  // Save
  const audioPath = `${outputDir}/${call.id}.mp3`;
  fs.writeFileSync(audioPath, buffer);
  
  return audioPath;
}
```

## Batch Download Script

```javascript
const fs = require('fs');

async function downloadAllRecordings(calls, apiKey, accountId) {
  const outputDir = './audio';
  fs.mkdirSync(outputDir, { recursive: true });
  
  const withRecording = calls.filter(c => c.recording && c.duration > 5);
  console.log(`Downloading ${withRecording.length} recordings...`);
  
  for (let i = 0; i < withRecording.length; i++) {
    const call = withRecording[i];
    console.log(`[${i + 1}/${withRecording.length}] ${call.id}`);
    
    try {
      await downloadRecording(call, apiKey, outputDir);
    } catch (err) {
      console.error(`  Failed: ${err.message}`);
    }
    
    // Rate limit
    await new Promise(r => setTimeout(r, 500));
  }
}
```

## Notes

- Recordings are MP3 format
- Signed URLs expire (get fresh URL before each download)
- File sizes range from 100KB to 5MB depending on duration
- Consider cleaning up audio files after transcription to save space
