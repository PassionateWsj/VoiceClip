# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

**Paused.** macOS built-in dictation (System Settings → Keyboard → Dictation, Fn ×2) provides better UX than a browser PWA for voice-to-text. Code and GitHub repo kept for future reference.

## Architecture

Single-file PWA (`index.html`) that runs Whisper ONNX models in-browser via Transformers.js for local speech-to-text. No backend, no API calls — model inference happens in WASM.

- `index.html` — self-contained app (UI + recording + transcription + PWA logic)
- `sw.js` — Service Worker caching the HTML and Transformers.js CDN library
- `manifest.json` — PWA manifest (add to iOS home screen)
- `clear.html` — cleanup page that nukes SW / IndexedDB / Cache API (iOS Safari's "Website Data" doesn't reach those)
- `vercel.json` — static site config, output dir `.`

## Key constraints learned

- **whisper-small (250MB) OOM on iOS Safari WASM.** Use `whisper-base` (75MB) or `whisper-tiny` (40MB) for mobile.
- **`getUserMedia` during `pointerdown` breaks gesture tracking.** Permission dialog steals focus, `pointerup` never fires. Separate permission from recording: one-tap enable → press-hold record.
- **Don't keep mic stream alive between recordings.** Call `getUserMedia` on each press and `stop()` tracks on release. After first grant, subsequent `getUserMedia` calls don't prompt.
- **Whisper base model outputs no punctuation and mixes traditional/simplified Chinese.** Tried `generate_kwargs:{suppress_tokens:[]}` — unverified fix. Proper solution would be Alibaba Cloud ASR API (free 3 months, ¥3.50/1k calls after).
- **Vercel deployment IPs (108.160.x.x) blocked in China.** `vercel.com` resolves to a different range and is reachable.

## Deploy

```bash
vercel --prod   # static site, no build step
```

Project was linked to `passionatewsjs-projects/voiceclip` on Vercel but has been removed. Re-link with `vercel link` if re-deploying.
