# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file, mobile-first Mahjong scoring app (Traditional Chinese UI). Tracks 番數 (fan count), settles rounds, maintains running 埋數 (accumulated scores), and syncs a shared 小氣簿 (ledger) to Firebase Realtime Database.

## Development

No build step, no dependencies, no package manager. Serve statically:

```bash
python -m http.server 3000
# or: npx serve .
# open http://localhost:3000/mahjong-scorer.html
```

Deploy: `vercel deploy` — `vercel.json` rewrites all routes to the HTML file.

## Architecture

Everything lives in `mahjong-scorer.html` — markup, CSS, and all JS in one file. No modules, no bundler.

- **Firebase**: v9 compat CDN SDK loaded from the HTML. Config in `firebase-config.js` (exposes `window.__FIREBASE_CONFIG__`). RTDB paths: `records/{timestamp}` and `players/{safeName}/history`.
- **State**: localStorage keys prefixed `mj_` (names, inputs, outputs, accumulated scores, undo snapshot). The app works offline; only the ledger requires network.
- **UI**: Three tab pages (番數記錄, 埋數, 小氣簿) switched via a bottom nav. Dark theme only, designed for 375–430px mobile viewports.
- **Scoring formula**: `ceil(r₀·1.5^(n-1) + r₁·1.5^(n-2) + … + rₙ₋₁·1.5⁰)` where rows compound from top to bottom.

## Conventions

- All UI text is Traditional Chinese (zh-Hant).
- Player names are sanitized (`.#$/[]` → `_`) before use as Firebase keys.
- Fonts: Noto Sans HK for Chinese, Inter for tabular numerics.
- CSS uses custom properties defined in `:root`. No utility framework.
