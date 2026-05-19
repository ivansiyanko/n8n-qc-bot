# Post-Production QC Bot: Automated Video Quality Checks

Automated quality control for exported videos â validates resolution, aspect ratio, duration, audio levels, codec, and platform-specific safe zones before delivery. Flags issues before they reach the client. Built with **n8n** and **FFmpeg**.

![n8n workflow](https://img.shields.io/badge/n8n-workflow-orange) ![QC Automation](https://img.shields.io/badge/QC-Automation-red) ![license MIT](https://img.shields.io/badge/license-MIT-green)

## What It Does

This n8n workflow runs comprehensive quality checks on video exports before they go to the media buying team or client:

- **Resolution Validation** â Checks width/height against platform minimums (600px Meta, 540px TikTok, 720px YouTube Shorts)
- **Aspect Ratio Check** â Validates 16:9, 9:16, 1:1, 4:5 against each target platform's accepted formats
- **Duration Limits** â Flags videos exceeding platform maximums (240s Meta, 180s TikTok, 60s YouTube Shorts)
- **Audio Level Analysis** â Detects mean volume, peak levels, and clipping using FFmpeg volumedetect
- **Frame Rate Validation** â Ensures FPS falls within platform-accepted ranges (24â60fps)
- **File Size Check** â Validates against upload limits per platform
- **Codec Compatibility** â Warns if codec isn't h264/HEVC for maximum platform compatibility
- **Safe Zone Awareness** â Reports platform-specific safe zones for text/UI elements
- **QC Score** â Calculates an overall quality score (0â100) based on checks passed
- **Pass/Fail Verdict** â Clear PASS or FAIL with detailed issue list

## Platform Specs Checked

| Platform | Max Duration | Min Width | Aspects | Max Size | Audio |
|----------|-------------|-----------|---------|----------|-------|
| **Meta** | 240s | 600px | 16:9, 9:16, 1:1, 4:5 | 4GB | Required |
| **TikTok** | 180s | 540px | 9:16, 1:1 | 500MB | Required |
| **YouTube Shorts** | 60s | 720px | 9:16 | 256MB | Required |

## How It Works

```
Webhook â Validate â Download Video â FFprobe Analysis
â Extract Metadata + Analyze Audio â Run Platform QC Checks
â Log to Sheet â Slack Report + Webhook Response
```

1. **Webhook Trigger** â Receives video URL, target platforms, project name, and client info
2. **Download Video** â Fetches the exported video file
3. **FFprobe Analysis** â Extracts all technical metadata (resolution, codec, fps, duration, bitrate)
4. **Audio Analysis** â Runs FFmpeg volumedetect to measure mean/peak audio levels and detect clipping
5. **Platform QC Checks** â Validates every metric against each target platform's specifications
6. **QC Log** â Appends results to Google Sheets for audit trail and historical tracking
7. **Slack Report** â Posts a formatted PASS/FAIL report with all issues, warnings, and metadata
8. **Webhook Response** â Returns the full QC report as JSON

## Quick Start

### Prerequisites
- n8n installed (self-hosted or cloud)
- FFmpeg and FFprobe installed on your n8n server
- Google Sheets credentials
- Slack webhook (optional)

### Setup
1. **Import the workflow** â Open n8n, go to Workflows â Import from File, select `workflow.json`
2. **Set up Google Sheet** â Create a sheet named "QC Log" with columns: Date, Project, File, Client, Verdict, Score, Issues, Warnings, Resolution, Duration, Aspect, FPS, Codec, Size MB, Audio dB
3. **Configure Slack** â Set the notification channel
4. **Activate the workflow** â Toggle to Active

### Usage

```bash
curl -X POST https://your-n8n-instance.com/webhook/qc-check \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "https://storage.example.com/exports/summer-sale-final.mp4",
    "file_name": "summer-sale-final.mp4",
    "project_name": "Summer Sale Campaign",
    "platforms": ["meta", "tiktok", "youtube_shorts"],
    "client_name": "Acme Corp",
    "delivery_type": "social_ad",
    "notify_channel": "#post-production"
  }'
```

## Example QC Report

```
FAIL (Score: 72/100)

Project: Summer Sale Campaign â summer-sale-final.mp4
1920x1080 (16:9) | 30fps | h264
0:32 | 24.5MB
-15.2dB mean / -3.1dB peak

Issues (2):
â¢ [TikTok] Aspect ratio 16:9 not supported. Required: 9:16, 1:1
â¢ [YouTube Shorts] Aspect ratio 16:9 not supported. Required: 9:16

Warnings (1):
â¢ [Audio] Mean volume -15.2dB slightly quiet (ideal: -16 to -14dB)

9 checks passed across meta, tiktok, youtube_shorts
```

## Customization

- **Add Platforms** â Extend `platformSpecs` with Snapchat, Pinterest, LinkedIn, Twitter/X specs
- **Custom Checks** â Add black frame detection, letterbox detection, or watermark verification
- **Batch Processing** â Loop over multiple files for bulk QC before campaign launch
- **Auto-Reject** â Connect to your project management tool to auto-reject failed deliverables
- **Historical Dashboard** â Build charts from the QC Log sheet to track quality trends over time

## License

MIT â free to use, modify, and share.

## Author

**Ivan Siyanko** â AI Automator
Website: [siyanko.com](https://siyanko.com) | GitHub: [@ivansiyanko](https://github.com/ivansiyanko)

---
Built with [n8n](https://n8n.io) â the workflow automation platform
