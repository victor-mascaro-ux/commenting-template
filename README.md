# comments-template

Starter template for Figma-style persistent comments over a Knomee HTML prototype. Copy this into any new prototype repo so every project gets pinned, saved feedback for free.

## What's in here

- `index.html` — the commenting shell. Wraps your prototype in an iframe, lets you drop a pin anywhere on it, and saves the comment thread to Firestore so it survives reloads and is visible to anyone with the link.
- `prototype.html` — a throwaway 3-screen dummy flow, only here to prove the commenting layer works. Replace it with your real prototype.

## Using this on a new prototype

1. Copy `index.html` into your new repo (keep the filename — GitHub Pages serves it as the site root).
2. Drop your real prototype file in as `prototype.html` (or point the iframe's `src` in `index.html` at whatever you name it), or delete the dummy screens and build your prototype directly in that file.
3. **Open `index.html` and change `PROJECT_ID`** near the top of the `<script>` block to something unique for this repo (e.g. the repo name). This ID is what keeps different prototypes' comments from colliding (see below). Never set it to `"state"`. If two deployed prototypes do end up sharing an ID, each will detect the other and show a persistent warning banner naming the conflicting site — there's no nag otherwise.
4. Commit and push. Comments persist automatically — nothing else to configure.

## How it works

- The prototype runs full-bleed with no app chrome around it — there's no header, toolbar, or "swap prototype" control. To use a different prototype, just replace `prototype.html` (or edit the iframe's `src`) and redeploy.
- Click the purple **speech-bubble FAB** (bottom-right), then click anywhere on the prototype to drop a pin. Each comment stores a screenshot of the prototype's state at that moment (via html2canvas), so pins stay meaningful even if the prototype has multiple screens.
- While placing, composing, viewing, or editing a comment, press **Esc** (or click the × ) to exit back to the live prototype.
- Comments sync through Firestore, with `localStorage` as a same-browser fallback if the network is unreachable.
- **Clear all** (in the sidebar header) deletes every comment at once, with a confirmation.

## Firestore setup — read this before deploying

This template writes to the **same Firebase project and the same `video-feedback` collection** used by the `knomee-video-repo` feedback tool. That's not a stylistic choice — this project's Firestore security rules only grant access to that one collection name, and there's no way to add a new one from here. So every prototype built from this template shares that collection tree, namespaced underneath it by `PROJECT_ID` as the document ID (`video-feedback/{PROJECT_ID}/comments/...`). The video tool itself always uses the doc ID `"state"`, which is why `PROJECT_ID` must never be `"state"` — and why every new prototype repo needs its own unique ID.

**Conflict detection**: every deployed site stamps its Firestore doc with its own URL on load. If a site finds another deployment's stamp under its `PROJECT_ID`, it shows a persistent (click-to-dismiss) warning naming the conflicting site, and re-stamps so the other side gets warned on its next load too. The warnings only stop once one of them changes its `PROJECT_ID`. localhost is exempt, so local dev against deployed data never triggers it.

If the Firestore rules ever change (e.g. a real `prototype-feedback` collection gets whitelisted), update the `db.collection('video-feedback')` calls in `index.html` accordingly — search for the comment above the `firebaseConfig` block.

## Testing the template itself

Serve this folder locally (e.g. `python3 -m http.server`) and open `index.html`. It loads `prototype.html` by default — click through its screens and leave a few comments to confirm pins, persistence, and reload all work before wiring in a real prototype.
