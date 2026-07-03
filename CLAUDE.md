# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **published mirror**, not the source of truth. It serves the EarlBear marketing site as
static HTML via **GitHub Pages** (off `main`, root path) at **https://discover.earlbear.com**.

The site is *authored elsewhere* — in the internal `EarlBear/sad-sandbox` repo under
`EarlBear/LandingPage/`. This repo holds only the two files that get served plus the Pages/CI/sync
plumbing. There is **no build step, no framework, no dependencies**: `index.html` and
`onboarding.html` are each a single self-contained file (inline HTML + CSS + JS).

## The most important rule: don't hand-edit the served HTML here

`index.html` and `onboarding.html` are **generated copies**. Edits made directly in this repo will be
**silently overwritten** on the next sync. To change the site content, change it in `sad-sandbox` and
then sync. Only touch the HTML here if you are deliberately hotfixing and understand it will diverge
from source until sad-sandbox is updated to match.

## Syncing from source

Use the **`sync-landing`** skill (`.claude/skills/sync-landing/SKILL.md`) — trigger it by asking to
"sync the landing page" / "pull latest from sad-sandbox". It:

1. Pulls the local source clone at `../sad-sandbox` (falls back to `gh api` if the clone is absent).
2. Diffs `EarlBear/LandingPage/` between the last-synced SHA and HEAD, and **shows the delta before
   copying**.
3. Copies **only** `index.html` + `onboarding.html` (never the methodology `.md` docs, never this
   repo's `CNAME`/`.nojekyll`/`README.md`/`.sync-source.json`).
4. Rewrites `.sync-source.json` and commits with a `Synced-from: sad-sandbox@<sha>` trailer.

**`.sync-source.json`** is the machine-readable record of exactly which sad-sandbox commit the
current HTML came from (`last_synced_sha`). It is the single source of truth for "how far behind are
we" — read it before syncing, update it after. The copy scope is intentionally an explicit file list,
not a whole-folder mirror; if source ever adds a served asset (e.g. an image), update both the
`files` list in `.sync-source.json` and the skill's copy step.

## Conventions (project-specific)

- **Never auto-push.** Commit locally, then stop and ask for explicit confirmation before any
  `git push` — this applies to manual work and to the `sync-landing` skill. The repo is public and
  pushing publishes; the user reviews first.
- **Never hand-edit `CNAME`'s value casually.** It drives the GitHub Pages custom domain. GitHub also
  auto-commits `CNAME` changes when the domain is set in the Pages UI, so this file can diverge
  between local and remote — reconcile by rebasing, and preserve a trailing newline.

## Serving / operations

- **GitHub Pages**: source is `main` / `/` (root). `.nojekyll` makes Pages serve files verbatim (no
  Jekyll). Custom domain is managed via the `CNAME` file + the Pages API.
  - State: `gh api repos/EarlBear/landing/pages --jq '{status, cname, https_enforced, cert: .https_certificate.state}'`
  - Enforce HTTPS (only works once the cert is issued):
    `gh api -X PUT repos/EarlBear/landing/pages -f cname=discover.earlbear.com -F https_enforced=true`
- **DNS** is on Cloudflare: `discover` is a CNAME → `earlbear.github.io` (DNS-only / grey cloud so
  GitHub's Let's Encrypt HTTP-01 validation can reach the origin). The apex/`www` records point at the
  Shopify store and must stay untouched. Note: the site's logo is loaded from the Shopify CDN on the
  apex — keep that in mind before any apex change.

## Secret scanning (public repo)

Four layers, three currently active:
- GitHub native secret scanning + push protection (server-side, enabled).
- Gitleaks **pre-commit hook** (`.pre-commit-config.yaml`) — run `pre-commit install` once per clone.
- Gitleaks **CI** (`.github/workflows/gitleaks.yml`) — a push/PR backstop. (May be blocked if GitHub
  Actions is disabled by an account/org billing lock — that's a billing issue, not a scan failure.)
- Ad-hoc scan: `gitleaks git .` (history) and `gitleaks dir .` (working tree).

## Local preview

No build. Serve the root and open both pages:

```bash
python3 -m http.server 8080
# http://localhost:8080/  and  http://localhost:8080/onboarding.html
```
