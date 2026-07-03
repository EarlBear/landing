---
name: sync-landing
description: Sync this repo's published landing page (index.html, onboarding.html) with the latest from the EarlBear/sad-sandbox source repo. Use when the user says "sync the landing page", "pull latest from sad-sandbox", "update the landing page", "check for landing page changes", or wants to see what changed upstream. Tracks the last-synced sad-sandbox commit in .sync-source.json, shows a diff before copying, and NEVER auto-pushes.
---

# Sync landing page from sad-sandbox

## Purpose

This repo (`EarlBear/landing`) is a **published mirror** served via GitHub Pages. The site is
authored in `EarlBear/sad-sandbox` under `EarlBear/LandingPage/`. This skill pulls the latest
source, shows what changed since the last sync, copies the served files over, records exactly which
source commit we synced from, and commits — **stopping to ask before any push**.

## Guardrails (read first)

- **Only these files are copied** from source: `index.html`, `onboarding.html`. Nothing else.
- **Never overwrite** `CNAME`, `.nojekyll`, `README.md`, or `.sync-source.json` from the source
  repo — those belong to this repo.
- **Never auto-push.** Commit locally, then STOP and ask the user for explicit push confirmation
  (use the AskUserQuestion tool). Push only after they confirm that specific push.
- **Always show the diff before copying** so the user sees what's coming in.
- The methodology docs (`DESKTOP.md`, `MOBILE.md`, `METHODOLOGY.md`) stay in sad-sandbox — do NOT
  copy them here.

## State file: `.sync-source.json`

Machine-readable record of the last sync, at the repo root:

```json
{
  "source_repo": "EarlBear/sad-sandbox",
  "source_path": "EarlBear/LandingPage",
  "source_clone": "../sad-sandbox",
  "files": ["index.html", "onboarding.html"],
  "last_synced_sha": "<full sha>",
  "last_synced_short": "<short sha>",
  "last_synced_commit_date": "<ISO date of the source commit>"
}
```

## Workflow

Run these steps from the repo root (`EarlBear/landing`).

### 1. Locate & update the source

Prefer the local clone recorded in `.sync-source.json` (`source_clone`, default `../sad-sandbox`):

```bash
SRC=$(python3 -c "import json;print(json.load(open('.sync-source.json'))['source_clone'])")
git -C "$SRC" status --short          # warn the user if the clone has uncommitted changes
git -C "$SRC" fetch origin
git -C "$SRC" pull --ff-only          # report if this fails (diverged/dirty)
```

**Fallback if the clone is missing** (`$SRC` doesn't exist): fetch the two files directly from
GitHub with the `gh` CLI instead of a local clone —
`gh api repos/EarlBear/sad-sandbox/contents/EarlBear/LandingPage/index.html --jq '.content' | base64 -d > index.html`
(and the same for `onboarding.html`), and get the current source SHA with
`gh api repos/EarlBear/sad-sandbox/commits/main --jq '.sha'`. Then skip to step 4.

### 2. Read the last-synced commit

```bash
LAST=$(python3 -c "import json;print(json.load(open('.sync-source.json'))['last_synced_sha'])")
NEW=$(git -C "$SRC" rev-parse HEAD)
```

If `LAST == NEW`, tell the user the landing page is already up to date and stop.

### 3. Show the delta (BEFORE copying)

Show the user what changed in the landing files since the last sync:

```bash
git -C "$SRC" log --oneline "$LAST..$NEW" -- EarlBear/LandingPage
git -C "$SRC" diff --stat "$LAST..$NEW" -- EarlBear/LandingPage
```

If they want detail on a specific file, show the full `git -C "$SRC" diff "$LAST..$NEW" -- <path>`.
Let the user confirm they want to proceed with the copy.

### 4. Copy the served files

```bash
cp "$SRC/EarlBear/LandingPage/index.html" ./index.html
cp "$SRC/EarlBear/LandingPage/onboarding.html" ./onboarding.html
```

### 5. Update `.sync-source.json`

Rewrite it with the new SHA. Get the fields from the source repo (do NOT invent a timestamp — read
the source commit's date):

```bash
NEW_SHORT=$(git -C "$SRC" rev-parse --short "$NEW")
NEW_DATE=$(git -C "$SRC" show -s --format='%cI' "$NEW")
```

Write those into `last_synced_sha` / `last_synced_short` / `last_synced_commit_date`, keeping the
other fields unchanged.

### 6. Verify locally (optional but recommended)

```bash
python3 -m http.server 8080   # open http://localhost:8080/ and /onboarding.html, then stop the server
```

### 7. Commit — then ASK before pushing

Review, then commit locally with a `Synced-from` trailer:

```bash
git add index.html onboarding.html .sync-source.json
git commit -m "Sync landing page from sad-sandbox@$NEW_SHORT

<one-line summary of what changed, from the log in step 3>

Synced-from: sad-sandbox@$NEW"
```

Then **STOP**. Use the AskUserQuestion tool to ask whether to push to `origin`. Push only on
explicit confirmation:

```bash
git push origin main    # ONLY after the user explicitly confirms this push
```

## Notes

- GitHub Pages rebuilds automatically on push to `main` (usually live within a minute or two).
- If the source ever adds new files that must be served (e.g. an image), update the `files` list in
  `.sync-source.json` and this skill's copy step — the scope is intentionally explicit, not a
  whole-folder mirror.
