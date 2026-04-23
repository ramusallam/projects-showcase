# Projects Showcase

Public-facing grid of Ramsey's Class Apps. Renders cards from `projects.json` and links each card to its live Vercel deployment. Built as plain static HTML/CSS/JS with no build step so it drops into Weebly via iframe without ceremony.

## Files

- `index.html` — shell page, fetches `projects.json` at runtime and renders cards alphabetically
- `styles.css` — visual design (populated by aesthetic pass)
- `projects.json` — the manifest that drives the grid
- `showcase-allowlist.json` — curated list of Class Apps folder names approved for the public showcase; consumed by the sync script
- `vercel.json` — static site config

## Embed URL for Weebly

```
https://projects-showcase.vercel.app
```

Drop that URL into a Weebly embed/iframe block.

## How to update

Do not hand-edit `projects.json`. The sync script (separate repo / tool) regenerates it from the allowlist plus each app's Vercel project metadata. To add or remove an app from the showcase:

1. Edit `showcase-allowlist.json` (add or remove the folder name)
2. Run the sync script
3. Commit the regenerated `projects.json`
4. Deploy: `vercel --prod` first, then `git push`

## Deploy order

Per Ramsey's rule: `vercel --prod` first, then `git push`. GitHub does not auto-deploy to Vercel on this private account.
