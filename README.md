# Spark Homes — Repair Estimator (PWA)

A mobile-first, offline-capable repair cost estimator for Spark Homes' acquisition team.
Agents walk a property on their phone, check off needed repairs room by room, snap photos
of equipment serial plates, and export a ZIP (Excel breakdown + photos) before submitting an offer.

**Single-file app, no build step, no server required.** Everything runs in the browser and
persists to `localStorage`.

---

## Run it locally

The app is a static site. Because service workers and the camera require a secure context,
serve it over `http://localhost` (file:// will not register the service worker):

```bash
# any static server works — pick one:
python3 -m http.server 8080
# or
npx serve .
```

Then open **http://localhost:8080** and, for the real experience, open it on your phone
(same network) or deploy the folder to any static host (GitHub Pages, Netlify, Vercel, S3).

To **install as a PWA**: open in Chrome (Android) → menu → *Add to Home screen*, or
Safari (iOS) → Share → *Add to Home Screen*. It launches standalone and works fully offline
after the first load.

### Files
| File | Purpose |
|------|---------|
| `index.html` | The entire app — markup, CSS, and JS in one file |
| `sw.js` | Service worker (offline app shell + cached CDN libs) |
| `manifest.json` | PWA manifest (installability, standalone display) |
| `icon.svg` / `icon-maskable.svg` | App icons |

---

## Approach

The app is **data-model driven** rather than hard-coded screens, which keeps the room/group
structure flexible (a core requirement — every house is different).

- **`CATALOG`** — all 108 line items from `Pricing List.csv`, keyed by `id` (the single source of truth).
- **`GROUPS`** — the 19 reference repair groups (plus 3 helpers: *General/Misc*, *Lighting*, *Closet*
  for the decoupled Living/Common and Bedroom room types), each mapping to a set of catalog ids.
- **`ROOM_TYPES`** — 7 room types (Interior/General, Kitchen, Bathroom, Bedroom, Living/Common,
  Systems & Structure, Exterior). Each references the groups relevant to it. Bathrooms/bedrooms/
  living areas auto-number as instances ("Bathroom 1", "Bathroom 2", …).

A project stores room instances + per-group selections, quantities, per-project price overrides,
photos, notes, and deal inputs. Totals and progress are recomputed and patched into the DOM in
place (no full re-render on quantity edits) so typing on a phone stays smooth.

### Pricing model (two layers)
1. **Global prices** (Menu → Pricing) edit the standard list for *all* projects, stored separately.
2. **Per-project override** (tap any line-item price) applies only to the current project and wins
   over the global value. CSV import/export round-trips the global list.

### Progress
Counted per group across every room instance — a group is complete when any item is checked
**or** it's explicitly marked *No action needed*.

---

## Creative addition — Deal Analyzer

A dedicated tab that turns the repair estimate into a go/no-go decision on the spot. It pulls the
live repair total, applies the investor **70% rule** to suggest a Maximum Allowable Offer, and
computes estimated net profit and return on cost after holding and selling costs. An agent
standing in the house immediately knows whether the numbers work. See `WRITEUP.html` for why.

## AI feature — Smart Add (on-device semantic search)
Tap **✨ AI Smart Add** and describe a problem in plain English ("the AC outside is dead",
"carpet is ruined", "full bathroom gut"). An on-device neural model
([TensorFlow.js Universal Sentence Encoder](https://www.tensorflow.org/hub)) embeds the phrase and
ranks the closest matching line items out of all 108, then one tap drops each item into the correct
room and group. This is real semantic matching, not keyword search, so "AC unit outside" finds
*Condensing Unit* even though those words never appear in the item name.

The model is **lazy-loaded only when the feature is opened** (so startup stays fast and the install
stays light) and the app **falls back to keyword search** if the model hasn't loaded or the device
is offline on first use — so the feature never feels broken. I chose this over photo OCR because
agents work in *empty* distressed houses where there's often little for a vision model to see,
whereas plain-language item lookup speeds up the core task in any house.

---

## Libraries (all via CDN, cached by the service worker for offline use)
- [SheetJS / xlsx](https://sheetjs.com) — Excel generation
- [JSZip](https://stuk.github.io/jszip/) — bundles the spreadsheet + photos into one ZIP
- [TensorFlow.js + Universal Sentence Encoder](https://www.tensorflow.org/js) — on-device semantic
  search for AI Smart Add (lazy-loaded on first use)

No frameworks — vanilla HTML/CSS/JS. CSS is hand-written (no Tailwind) so the UI is fully
self-contained and reliable offline.

## Known limitations
See `WRITEUP.html` ("what is fragile") for the honest list.
