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
- [x] Commit + push `gh-pages`
- [x] GitHub Pages already enabled by the fork — confirmed source = `gh-pages` root,
      `cname: null`, builds are `legacy`
- [x] Verify the URL serves 200 and contains both injected blocks

## Bug I introduced, and the fix (commit `687df78`)
First injection commit (`7bcb347`) computed `lastIndexOf('</head>')` and
`lastIndexOf('</body>')` up front, then inserted the CSS first. That insertion
shifted everything after `</head>`, so the stale `</body>` index pointed into
the middle of the inline webpack runtime — the touchmove `<script>` spliced
`function` into `fu` + `nction(){...}` and the app could not boot.

Fixed by inserting **inside-out**: place the later `</body>` block first, then
recompute and place the `</head>` block. Verification script
(`verify.js`) strips the two blocks one at a time, recomputing indices after
each removal, and md5-compares the remainder against upstream.

## Verification results (commit `687df78`)
- `GET /wick-editor/` → HTTP 200, 7857 bytes
- Stripped page md5 = `a45bec1005baadc07099ef52e0f596e2` = upstream `index.html`
  → page is upstream plus **exactly** the two injected blocks, nothing else touched
- `</style>\n</head>` present → style block sits immediately before `</head>`
- `</script>\n</body>` present → touchmove script sits immediately before `</body>`
- Inline webpack runtime ends cleanly: `...var s=n;a()}([])</script><script src="./static/js/1...`
- All 10 referenced assets return 200/206 (main/1/runtime chunks, both CSS,
  wickengine.js, preloadjs, gif.js, favicon, manifest)

## Not verified
- No visual/JS boot check: no desktop browser is connected to this session, so
  rendering could not be inspected. Structure + assets are proven; the actual
  first paint on a phone still needs a human eyeball.
