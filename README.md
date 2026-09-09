# Before / After Splat Comparison

A single static page that shows two Gaussian Splat scenes (e.g. a landscape
before and after a building development) side by side with a draggable
divider, using one shared camera so you can fly/orbit around while comparing.

Built with [Three.js](https://threejs.org) + [Spark](https://sparkjs.dev)
(a fast Gaussian Splatting renderer for Three.js). No build step — it's
plain HTML/JS loaded from a CDN, so it runs directly on GitHub Pages.

## 1. Add your splat files

Export both scenes from PostShot (File → Export → Splat, or similar — PostShot
supports `.ply` directly). Put them in the `splats/` folder:

```
splats/
  before.spz   (or before.ply / before.ksplat)
  after.spz
```

Then update the two filenames at the top of `index.html` if you used
different names:

```js
const BEFORE_URL = "splats/before.spz";
const AFTER_URL  = "splats/after.spz";
```

**Compress first if you can.** Raw `.ply` exports from PostShot are often
several hundred MB to a few GB, which is slow to load over the web and can
hit GitHub's file-size limits. Convert to `.spz` or `.ksplat` (10–20x
smaller, same visual quality) with a free tool before uploading:

- [PlayCanvas SuperSplat](https://playcanvas.com/supersplat/editor) — drag
  and drop your `.ply`, clean up stray floaters, export as `.spz` or
  compressed `.ply`. Also useful for trimming each scene down to just the
  area you want to showcase.
- [Spark's CLI conversion tools](https://sparkjs.dev/docs/) if you prefer a
  command line.

**Alignment matters.** This viewer uses one shared camera for both scenes,
so the before/after comparison only looks right if both splats share the
same origin and orientation. If PostShot exported them already aligned
(e.g. both reconstructed against the same GPS/reference points), you're set.
If not, nudge one of them into place using the `BEFORE_TRANSFORM` /
`AFTER_TRANSFORM` objects near the top of `index.html` (position, rotation
in radians, uniform scale) — SuperSplat can also help you find the right
offset visually before you bake it in.

## 2. Try it locally

Browsers block `file://` module loading, so serve the folder over HTTP:

```bash
cd splat-compare
python3 -m http.server 8000
```

Open `http://localhost:8000`.

## 3. Publish to GitHub Pages

```bash
git init
git add .
git commit -m "Splat before/after comparison"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```

Then in the repo: **Settings → Pages → Source → Deploy from a branch →
`main` / root**. Your site will be live at
`https://<you>.github.io/<repo>/` within a minute or two.

### If your splat files are large

- GitHub blocks pushes with any single file over 100 MB, and warns above
  50 MB. If your compressed `.spz` files are still large, use
  [Git LFS](https://git-lfs.com/):
  ```bash
  git lfs install
  git lfs track "splats/*.spz" "splats/*.ply" "splats/*.ksplat"
  git add .gitattributes
  ```
  Note: GitHub Pages *does* serve LFS-tracked files correctly, but LFS
  bandwidth is limited on free accounts — fine for personal sharing, worth
  checking if you expect heavy traffic.
- Alternatively, host the two splat files elsewhere (e.g. an S3/R2 bucket
  or Cloudflare Pages, which have no such limit) and point `BEFORE_URL` /
  `AFTER_URL` in `index.html` at the full `https://` URLs instead of a
  local path. The CDN they're hosted on needs to allow cross-origin
  requests (CORS) for the browser to fetch them.

## How it works

- Both splat scenes are loaded into the same Three.js scene at the same
  position, and one `OrbitControls` + WASD rig drives a single shared
  camera — so orbiting or flying through the scene moves both "sides"
  identically.
- Each frame is rendered twice with `renderer.setScissor()`: once with only
  the "before" mesh visible, clipped to the pixels left of the divider,
  and once with only the "after" mesh visible, clipped to the pixels right
  of it. Dragging the circular handle just moves that clip boundary, which
  is why the comparison stays pixel-accurate at any camera angle.

## Controls

- **Drag** — look around
- **Scroll / pinch** — zoom
- **Right-click drag** (or two-finger drag on trackpad) — pan
- **W A S D** — fly forward/back/strafe, **Q/E** — down/up, hold **Shift**
  to move faster
- **Drag the circular handle** — move the before/after divider
