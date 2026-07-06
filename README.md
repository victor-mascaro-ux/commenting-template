# comments-template

A persistent, Figma-style commenting overlay ("Neutral Frost") that sits on top of any throwaway Knomee HTML prototype. Reviewers toggle comment mode, click anywhere on the frame to drop a numbered pin, and leave/reply to notes that persist across reloads and across screens. Copy this into any new prototype repo.

## What's in here

- `index.html` — the overlay shell. Wraps your prototype in an iframe and layers the frosted-glass comment chrome on top. All styles are scoped under a `cc-` prefix (and the prototype lives in its own iframe), so nothing leaks in either direction.
- `prototype.html` — a throwaway 3-screen dummy flow, only here to prove the overlay works. Replace it with your real prototype.

## Using this on a new prototype

1. Copy `index.html` into your new repo (keep the filename — GitHub Pages serves it as the site root).
2. Drop your real prototype in as `prototype.html` (or point the iframe's `src` at whatever you name it).
3. **Change the config constants** at the top of the `<script>` block:
   - `PROJECT_ID` — **required**, unique per repo (e.g. the repo name). This is the Firestore namespace that keeps different prototypes' comments from colliding. Never `"state"`. If two deployed prototypes do end up sharing an ID, each detects the other and shows a persistent warning naming the conflicting site.
   - `PROTO_TITLE` — the title shown in the comment-mode top bar.
   - `THEME` — `'neutral-frost'` (default) | `'smoke-tray'` | `'branded-frost'`.
   - `SEED_DEMO` — leave `false`. Set `true` only to preview the overlay itself: it skips all persistence and seeds 3 demo comments with one thread open.
4. Commit and push. Comments persist automatically — nothing else to configure.

## Design system — "Neutral Frost"

The chrome deliberately does **not** reuse Knomee's brand palette, because it floats over prototypes that have their own design systems. It reads as a neutral, translucent instrument sitting *above* the work:

- Near-monochrome ink + greys + white frosted glass (`backdrop-filter: blur`), so the prototype's color shows through faintly.
- Chrome typography is Plus Jakarta Sans — intentionally not the prototype's font — with tabular figures for counts/timestamps.
- The only full-strength brand color is the tiny teal Knomee "K" chip in the top bar.
- **Pins are teardrop map-pins** (not circles): solid ink fill, white number, white halo + soft shadow — verified legible on both white and dark/saturated backgrounds. The tip anchors on the exact clicked coordinate; positions are stored as percentages so pins survive resizes.
- **Thread popovers carry no identity chrome** — no avatars, no names. Just timestamp, resolve/delete, the notes, and a reply box.

### Theme presets (config, not separate code paths)

- **`neutral-frost`** — white frosted glass, zero brand color in the chrome.
- **`smoke-tray`** — dark translucent chrome; the rail detaches and floats (inset, rounded), the top bar becomes a floating pill. Pins unchanged.
- **`branded-frost`** — Neutral Frost plus a faint purple wash, purple count chip / active-card bar / active-pin ring / FAB tint, and compact single-line rail rows.

## How it works

- The **FAB** (bottom-right, frosted, chat-bubble icon) toggles comment mode. Outside comment mode only the FAB and existing pins are visible and the prototype is fully interactive.
- In comment mode: frosted **top bar** (K chip, title, hint, × to exit), frosted **comments rail** on the left (cards with thumbnail, numbered pin token, timestamp, one-line snippet — newest first), and a **"Press Esc to exit comment mode"** pill.
- **Click anywhere on the frame** to drop a numbered teardrop pin and open an empty thread with focus in the input. The first note creates the thread (closing without typing discards the pin). A thumbnail of the frame is captured (html2canvas) for the rail card.
- **Enter** sends; **Shift+Enter** inserts a line break. Clicking a pin or rail card opens its thread; clicking elsewhere closes it; **Esc** closes the thread first, then exits comment mode.
- **Resolve** (check) hides the pin but keeps the thread stored with `resolved: true` — nothing you didn't write is ever cleared. **Delete** and **Clear all** remove threads for real (with confirmation).
- **Multi-screen:** pins belong to the screen they were dropped on. The overlay reads the prototype's current screen without integration work: expose `window.__ccScreenId` in your prototype, or use the `.screen.active` element-id convention (like the dummy). Prototypes with neither get a single `default` screen. Legacy comments without a screen show on every screen.

## Persistence — deliberate deviation from the design spec

The design spec called for `localStorage` persistence. This template instead keeps **Firestore as the source of truth** (with `localStorage` as a fast-paint seed and offline fallback), because comments need to be shared across reviewers and devices — same behavior as the knomee-video feedback tool.

It writes to the **same Firebase project and the same `video-feedback` collection** as that tool. That's not a naming choice: the project's Firestore security rules only allow that one collection name, and there's no console access to add another. Every prototype built from this template shares that collection tree, namespaced by `PROJECT_ID` as the document ID (`video-feedback/{PROJECT_ID}/comments/...`). The video tool itself uses the doc ID `"state"`, which is why `PROJECT_ID` must never be `"state"`.

If the Firestore rules ever change (e.g. a real `prototype-feedback` collection gets whitelisted), update the `db.collection('video-feedback')` calls in `index.html` — search for the comment above the `firebaseConfig` block.

## Testing the template itself

Serve this folder locally (e.g. `python3 -m http.server`) and open `index.html`. It loads `prototype.html` by default — toggle comment mode, drop pins on a light and a dark area (legibility check), reply, resolve, and reload to confirm persistence. Or set `SEED_DEMO = true` for an instant no-network preview with seeded comments.
