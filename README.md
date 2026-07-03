# EarlBear — Landing Page (published)

This repo publishes the EarlBear marketing site as **static HTML via GitHub Pages** off the `main`
branch, served at **https://landing.earlbear.com**.

It is a **published mirror**, not the source of truth. The site is authored in the internal
[`EarlBear/sad-sandbox`](https://github.com/EarlBear/sad-sandbox) repo under
`EarlBear/LandingPage/`. This repo holds only what needs to be served:

| File | Purpose |
|---|---|
| `index.html` | Main marketing site (single-file: inline HTML + CSS + JS). |
| `onboarding.html` | Multi-step onboarding flow (single-file). |
| `CNAME` | Custom domain for GitHub Pages: `landing.earlbear.com`. |
| `.nojekyll` | Tells Pages to serve files as-is (no Jekyll processing). |
| `.sync-source.json` | Records the exact `sad-sandbox` commit these files were synced from. |

Both HTML files are fully self-contained. Their only external references are Google Fonts and the
EarlBear logo (served from the Shopify CDN on `earlbear.com`).

## Keeping it in sync

The source is actively iterated in `sad-sandbox`. To pull the latest, use the **`sync-landing`**
skill (`.claude/skills/sync-landing/`) via Claude Code — just ask to *"sync the landing page"*. It:

1. Pulls the latest `../sad-sandbox` clone.
2. Shows exactly what changed in `LandingPage/` since the last sync (from `.sync-source.json`).
3. Copies `index.html` + `onboarding.html` over.
4. Updates `.sync-source.json` and commits with a `Synced-from: sad-sandbox@<sha>` trailer.
5. Stops and asks before pushing — it never auto-pushes.

## Running locally

No build step:

```bash
python3 -m http.server 8080
# then open http://localhost:8080/ and http://localhost:8080/onboarding.html
```

## DNS

`landing.earlbear.com` is a Cloudflare `CNAME` → `earlbear.github.io`. The apex/`www` records
(pointing at the Shopify store) are unaffected.
