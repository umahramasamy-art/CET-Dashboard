# CET Market Intelligence Dashboard — deployment guide

## What's in this package
- `index.html` — the dashboard page (static HTML/CSS/JS, no build step needed)
- `data.json` — all the programme, competitor, trend, funding, gap, partnership and NYP module data. The page fetches this file on load and on every "Refresh data" click.

The page and the data are deliberately separate: publishing an update just means replacing `data.json` — you never need to touch or re-deploy `index.html` again.

## Option A: GitHub Pages (recommended — free, simple, works well with Claude Cowork)

1. Create a new GitHub repository (public or private — Pages works with both on paid plans; public repos get Pages free).
2. Upload `index.html` and `data.json` to the root of the repo (or a `/docs` folder — either works, just match it in step 3).
3. In the repo, go to **Settings → Pages**, set the source branch (e.g. `main`) and folder (`/root` or `/docs`), save.
4. GitHub will give you a live URL, typically `https://<your-username>.github.io/<repo-name>/`.
5. Every time `data.json` in the repo is updated and pushed, the live page reflects it on next load — no rebuild step.

## Option B: Netlify / Vercel / S3

Any static host works the same way — just make sure `index.html` and `data.json` sit in the same folder (same-origin), so the browser's `fetch('./data.json')` call succeeds without CORS issues. Drag-and-drop deploy (Netlify) or `aws s3 sync` (S3) both work fine with this two-file structure.

## Setting up automatic refreshes with Claude Cowork

Claude Cowork supports scheduled tasks that can run on a recurring cadence or on demand. To automate this dashboard's refresh:

1. Open Claude Cowork and connect a GitHub connector (or whichever connector matches your chosen host) with write access to the repo/bucket hosting this dashboard.
2. Create a scheduled task with a prompt along these lines:

   > Re-scan the Singapore CET landscape for Customer Experience, Relationship Management, Marketing and Service Excellence programmes across Polytechnics, Universities, and ITE (including Work-Study Diplomas). Compare findings against the current `data.json` in [repo/path]. Update programme entries that have changed (fees, funding, dates), add any genuinely new programmes using the same schema, and update `meta.lastUpdated` to the current timestamp. Preserve the existing `nypModuleRegistry`, `nypModuleMap`, and `rationaleMap` structure — only add new entries there if a new programme needs module-level benchmarking against NYP's Diploma in Business Practice (CX&RM). Commit the updated `data.json` to the repo.

3. Set the cadence (e.g. monthly), or leave it as an on-demand task you trigger manually whenever you want a refresh.
4. Each run overwrites `data.json` in place — the published page picks up the change automatically on next load, with no redeploy needed.

## data.json schema reference

```
{
  "meta": { "lastUpdated": ISO date string, "snapshotLabel": string, "scopeNote": string },
  "programmes": [ { name, provider, type, area, level, duration, fee, feeNum, funding, skills, overlap, overlapClass, overlapPct, cert, conf }, ... ],
  "nypModuleRegistry": [ { mc, mcName, module, hrs }, ... ],
  "nypModuleMap": { "<programme name>": ["<module name>", ...], ... },
  "rationaleMap": { "<programme name>": "<HTML string explaining the overlap %>", ... },
  "competitors": [ { name, role, points: [...], tags: [...] }, ... ],
  "trends": [ { h, p, s }, ... ],
  "funding": [ { h, p, s }, ... ],
  "gaps": [ { t, p }, ... ],
  "partners": [ { name, role, points: [...] }, ... ],
  "summaryThreats": [ { t, p }, ... ],
  "summaryOpps": [ { t, p }, ... ],
  "specs": [ { t, p }, ... ]
}
```

Any future refresh (manual or via Cowork) just needs to produce a valid JSON file matching this shape — `index.html` never needs to change.
