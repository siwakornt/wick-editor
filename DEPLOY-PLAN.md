# Deploy Wick Editor + mobile scroll-lock to GitHub Pages

Owner: siwakornt · Started: 2026-10-25

## Goal
A ready-to-use public URL running Wick Editor with the user's exact
touch-scroll-lock `<script>` and `html, body { ... }` CSS injected.

## Decisions (agreed / my call)
- **Host: GitHub Pages** — user asked for "ลิ้งพร้อมใช้งาน" (ready link).
- **Deploy from the existing `gh-pages` branch** (955 pre-built files).
  My call: the source stack is `react-scripts 2.0.5` + `node-sass ^4.9.4`,
  which cannot build on this device's Node 26 / Termux / arm64. Rebuilding
  would be a multi-hour fight with a dead toolchain for zero benefit —
  the branch IS the official build.
- **Repo: fork** `siwakornt/wick-editor` — keeps upstream source on GitHub
  (server-side copy, no 128 MB download) and lets us pull upstream updates.
- **Delete `CNAME`** — it points at `wickeditor.com`, a domain we do not own.
  Leaving it makes the Pages deployment fail domain verification.

## Steps
- [x] Check toolchain + GitHub auth (gh logged in as siwakornt, `repo`+`workflow`)
- [x] Inspect repo: default `master`, has built `gh-pages` branch
- [x] Fork → https://github.com/siwakornt/wick-editor
- [x] Shallow-clone branch `gh-pages` → `~/wick-editor`
- [x] Inject user's `<style>` before `</head>` and `<script>` before `</body>`
- [x] Delete `CNAME`
- [ ] Commit + push `gh-pages`
- [ ] Enable GitHub Pages (branch `gh-pages`, root) via API
- [ ] Verify the URL serves 200 and contains both injected blocks

## Verification bar
- `https://siwakornt.github.io/wick-editor/` returns HTTP 200
- Response body contains `overscroll-behavior: none` and the `touchmove` guard
- Static asset (`main.*.chunk.js`) referenced by index.html returns 200
- Optional: open in browser and confirm the page renders the editor
