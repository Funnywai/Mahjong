# 麻雀計分 · Mahjong Scorer

A single-file, mobile-first Mahjong scoring app. Track 番數, settle rounds, keep a running 埋數, and sync a shared 小氣簿 (ledger) to Firebase Realtime Database.

No build step. Drop the folder on any static host (Vercel works out of the box).

---

## What it does

### 番數記錄 — current round
- 3-column grid for the three opponents, each with 7 stackable input rows.
- Tap any cell to open a full-screen numpad (digits, ±, backspace, clear, confirm).
- Output is recalculated on every edit:

  ```
  output = ceil( r₀·1.5ⁿ⁻¹ + r₁·1.5ⁿ⁻² + … + rₙ₋₁·1.5⁰ )
  ```

  where `r₀…rₙ₋₁` are the filled cells from top to bottom. The most recent row carries weight 1; earlier rows compound.
- **踢半** — halves the displayed output (`floor(output / 2)`), inputs untouched.
- **找數** — settles the column: subtracts the output from that opponent's 埋數, adds it to 我, clears the column, and records a one-step undo snapshot.

### 埋數 — accumulated standings
- Diamond layout: 閒2 on top, 閒1 / 閒3 in the middle, 我 on the bottom.
- Scores color-code green / red / muted and bump-animate whenever they change.
- **寫入小氣簿** pushes the snapshot to Firebase.

### 小氣簿 — shared history
- Reads the most recent records from RTDB, grouped by date (newest first).
- Pull-to-refresh via the refresh button; skeleton loaders while fetching.

### FAB menu
- **重設牌局** — clears inputs, outputs, 埋數; keeps names.
- **改名** — rename all four seats (persists to localStorage).
- **復原** — reverts the last 找數 (inputs, outputs, and 埋數).

Everything except the ledger lives in `localStorage`, so the app survives refreshes and offline use.

---

## Stack

- Vanilla HTML / CSS / JS — zero dependencies, zero build.
- Firebase Realtime Database via the v9 **compat** CDN SDK.
- Noto Sans HK (Traditional Chinese) + Inter (tabular numerics) from Google Fonts.
- Dark theme only, tuned for 375–430px viewports.

---

## Project layout

```
.
├── mahjong-scorer.html   # the whole app — markup, styles, and logic
├── firebase-config.js    # exposes window.__FIREBASE_CONFIG__ (public RTDB config)
├── vercel.json           # rewrites every path to mahjong-scorer.html
└── README.md
```

---

## Run locally

Any static server will do:

```bash
python -m http.server 3000
# open http://localhost:3000/mahjong-scorer.html
```

or:

```bash
npx serve .
```

---

## Deploy to Vercel

```bash
vercel deploy
```

`vercel.json` rewrites all routes to the HTML file, so deep links still work.

---

## Firebase

The RTDB URL is hardcoded in `firebase-config.js`. Realtime Database web config is safe to expose — access control belongs in your **Database Rules**, not in client code.

### Operations the app performs

| Path | Op | Trigger |
|------|----|---------|
| `records` | `orderByChild('timestamp').limitToLast(200)` read | opening 小氣簿 |
| `records/{timestamp}` | set | 寫入小氣簿 |
| `players/{safeName}/history` | push | 寫入小氣簿 (per player) |

Player names are sanitized (`.#$/[]` → `_`) before being used as keys.

### Data shape

```json
{
  "records": {
    "1715200000000": {
      "date": "2026-05-09",
      "timestamp": 1715200000000,
      "players": { "我": 50, "閒1": 0, "閒2": -50, "閒3": 0 }
    }
  },
  "players": {
    "我": {
      "history": {
        "<pushId>": { "timestamp": 1715200000000, "date": "2026-05-09", "score": 50 }
      }
    }
  }
}
```

---

## localStorage keys

| Key | Meaning |
|-----|---------|
| `mj_names` | `[閒1, 閒2, 閒3, 我]` |
| `mj_inputs` | 3 columns × 7 cells (null = empty) |
| `mj_outputs` | current computed outputs (after any 踢半) |
| `mj_tikai` | whether each column's output has been halved |
| `mj_accumulated` | `[閒1, 閒2, 閒3, 我]` running totals |
| `mj_undo_snapshot` | last 找數 snapshot, or null |

---

## License

MIT.
