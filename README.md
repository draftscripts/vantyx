# VantyX — listing assets

Marketing / listing assets for the **VantyX** admin dashboard template (ThemeForest),
plus the source templates and docs to regenerate them. Images are served via the
jsDelivr CDN and embedded in the item description.

> **Zip files are not committed here** (`.gitignore` excludes `*.zip`). They're build
> outputs you upload to ThemeForest — see "Build the upload files" below.

## Layout

| Path | What |
|------|------|
| `previews/` | Final images — CDN-served, used in the item description & uploads |
| `raw/` | Raw app screenshots (1920×1280) captured from the running app |
| `cover.html`, `gallery.html`, `thumb.html`, `preview-frame.html` | HTML templates the images are rendered from (they reference `raw/`) |
| `THEMEFOREST-LISTING.md` | Full listing copy — title, key features, HTML description, tags, changelog, reviewer message, category/attributes, pricing |

## Images (`previews/`)

| File | px | Use |
|------|----|-----|
| `cover.png` | 2340×1560 | Cover / inline preview (description) |
| `thumbnail-590x300.png` | 590×300 | Theme-Preview main image (`01_…`) |
| `VantyX-Thumbnail-80x80.png` | 80×80 | Item Thumbnail |
| `VantyX-Feature-Dashboards-560x380.png` | 560×380 | Description feature card |
| `VantyX-Feature-Components-560x380.png` | 560×380 | Description feature card |
| `VantyX-Large-Preview-1180x2697.png` | 1180×2697 | Description tall preview |
| `preview-1.png … preview-9.png` | 2100×1400 | Gallery screens |

## CDN

```
https://cdn.jsdelivr.net/gh/draftscripts/vantyx@main/previews/<file>
```

Raw alternative: `https://raw.githubusercontent.com/draftscripts/vantyx/main/previews/<file>`.
Purge after replacing an image: `https://purge.jsdelivr.net/gh/draftscripts/vantyx@main/previews/<file>`
(or pin a version path like `@v1.0.0`).

## Regenerate the preview images

The previews are real app screenshots framed by the HTML templates.

1. **Run the app** (in the product repo):
   ```bash
   pnpm build && PORT=3100 node .output/server/index.mjs
   ```
2. **Capture the raw screenshots** at 1920×1280 into `raw/` (browser automation). For each
   shot, set cookies before navigating to force the state:
   - `auth_token=demo` (+ `user_data`) to pass the auth gate
   - `themeConfig` JSON for theme/variant/direction/menu, e.g.
     `{"theme":"dark",...}`, `{"themeVariant":"rose",...}`, `{"rtlClass":"rtl",...}`,
     `{"menu":"horizontal",...}`
   - Captured set: `dashboard-light, analytics-light, analytics-dark, customizer-light,
     horizontal-light, ecommerce-rose, crm-rtl, kanban-light, landing-light, ui-buttons`
3. **Render the templates to PNG** — serve this folder and screenshot at the right viewport:
   ```bash
   python3 -m http.server 8099       # serves cover.html / gallery.html / thumb.html + raw/
   ```
   - `cover.html`  → viewport **2340×1560** → screenshot → `previews/cover.png`
   - `gallery.html` → viewport **2100×1400** → screenshot each `#p1…#p9` element → `previews/preview-1..9.png`
   - `thumb.html`  → viewport **1180×600** → screenshot → `sips -z 300 590` → `previews/thumbnail-590x300.png`

## Build the upload files (the 3 things ThemeForest wants)

These are **not** committed here.

### 1. Main File(s) — the product zip
Built in the **product repo** (not this one):
```bash
pnpm zip          # → vantyx-v1.0.0.zip
```
`scripts/pack.sh` runs `git archive` of `HEAD`, so only tracked source ships; author-only
files (`CLAUDE.md`, `.claude/`, `scripts/`, `.github/`, `.vscode/`) are dropped via
`.gitattributes export-ignore`, and it aborts if any leak. **Commit first** (it archives HEAD).

### 2. Theme Preview — a ZIP of preview images (not HTML)
First image must be **590×300**, named `01_…`; the rest `02_…` in display order:
```bash
mkdir tp && cd tp
cp ../previews/thumbnail-590x300.png          01_vantyx-preview.png   # 590×300 (required first)
cp ../previews/cover.png                      02_vantyx-cover.png
cp ../previews/preview-1.png                  03_vantyx-dashboards.png
cp ../previews/preview-2.png                  04_vantyx-analytics.png
cp ../previews/preview-3.png                  05_vantyx-dark-mode.png
cp ../previews/preview-4.png                  06_vantyx-customizer.png
cp ../previews/preview-5.png                  07_vantyx-menu-layouts.png
cp ../previews/preview-6.png                  08_vantyx-color-themes.png
cp ../previews/preview-7.png                  09_vantyx-rtl.png
cp ../previews/preview-8.png                  10_vantyx-apps.png
cp ../previews/preview-9.png                  11_vantyx-landing.png
zip -rqX ../vantyx-theme-preview.zip .
```

### 3. Thumbnail
`previews/VantyX-Thumbnail-80x80.png` (80×80).

### Optional — self-hosted live demo (SPA)
ThemeForest hosts no live demo for this category, but you can host your own and link it.
Build a reload-safe static SPA (temporarily, in the product repo, then revert the config):
```ts
// nuxt.config.ts (temporary)
ssr: false,
router: { options: { hashMode: true } },   // /#/dashboard — reloads work on any static host
image:  { provider: 'none' },               // <NuxtImg> serves /images/* (no IPX server)
```
```bash
NUXT_PUBLIC_COOKIE_CROSS_SITE=true pnpm generate
cd .output/public && zip -rqX ../../vantyx-live-demo-spa.zip . -x '*.gz' '*.br'
git checkout -- nuxt.config.ts
```
Demo login: `demo@example.com` / `password`.
