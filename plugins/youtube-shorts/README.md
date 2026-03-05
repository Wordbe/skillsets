# youtube-shorts

YouTube Shorts video generator for Claude Code using Remotion + Playwright.

## Features

- Automated website screenshot capture via Playwright
- React-based video composition (1080x1920, 30fps)
- Royalty-free BGM from Pixabay
- MP4 rendering pipeline
- 5-scene structure optimized for 15-second shorts

## Usage

```
/youtube-shorts [topic or URL]
```

## Requirements

- Node.js, npm
- ffmpeg (`brew install ffmpeg`)
- Playwright (`npx playwright install chromium`)
