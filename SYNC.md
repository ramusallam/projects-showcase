# Showcase Sync Workflow

The showcase is curated, not automatic. Nothing appears on the public site
until Ramsey explicitly approves it. This doc describes the sync script that
keeps `projects.json` in step with the apps Ramsey has approved.

## What the sync script does

`../sync-showcase.sh` does five things, in order:

1. Scans `Class Apps/` for subfolders (skipping hidden folders and
   `projects-showcase` itself).
2. Reads `projects-showcase/showcase-allowlist.json`, a JSON array of folder
   names approved for public display.
3. For every allowlisted folder it reads `.vercel/project.json` to get the
   Vercel `projectName`, builds the live URL
   (`https://<projectName>.vercel.app`), and refreshes only that URL in
   `projects.json`.
4. Writes `projects.json` with a fresh `generated_at` timestamp and the
   `projects` array sorted alphabetically by slug.
5. Prompts to deploy. On yes it runs `vercel --prod` first, then
   `git add projects.json`, `git commit`, and `git push`.

The script never rewrites display names or descriptions. Those are sticky
by design because they carry Ramsey's voice.

## When to run it

Run `sync-showcase.sh` in any of these cases:

- You added a folder name to `showcase-allowlist.json` after approving a
  new app.
- You removed a folder from the allowlist because an app is being retired.
- An existing app got a new Vercel project name and the public URL needs
  to update.
- You just want to regenerate the timestamp and confirm everything still
  lines up.

## How to approve a new app

1. Confirm Ramsey has said yes to publishing it. Do not publish on your own.
2. Add the folder name to `showcase-allowlist.json` (the array of folder
   names, exactly as they appear in `Class Apps/`).
3. Add a new entry to `projects.json` under `projects` with the full
   shape used by the other entries: at minimum `slug` (matches the folder
   name), `name` (display name in Ramsey's voice), `description` (hand
   written, not generated), and any other fields the showcase app expects.
   Leave `url` blank or use a placeholder, the sync script fills it in.
4. Run `./sync-showcase.sh` from `Class Apps/`.
5. Answer `y` at the deploy prompt.

If you forget step 3 the script exits with a clear error and does not
write the manifest. That is on purpose, it prevents a blank card from
going live.

## How to remove an app

1. Remove the folder name from `showcase-allowlist.json`.
2. Optionally, delete that project's entry from `projects.json` (the sync
   script will not carry it forward once it is off the allowlist, but
   cleaning up keeps the file tidy).
3. Run `./sync-showcase.sh`.
4. Answer `y` to deploy.

## Deploy order, non negotiable

Vercel first, GitHub second. Always.

```
vercel --prod         # first
git push              # second
```

GitHub on Ramsey's private account does not auto deploy to Vercel. If you
push to GitHub before deploying to Vercel, production is stale until
someone deploys manually. The sync script already enforces the right
order, do not work around it.

## Troubleshooting

- `jq` missing: install with `brew install jq` and re-run.
- `[ERROR] ... missing from projects.json`: you added the folder to the
  allowlist but did not write a description yet. Do that and re-run.
- `[WARN] ... no .vercel/project.json found`: the app was never deployed,
  or `.vercel/` is gitignored and was wiped. Run `vercel link` inside
  that app folder, then re-run sync.
- `[SKIP] Not on allowlist: ...`: expected output. The script is telling
  you about a folder it found but chose not to publish. Only act on this
  if Ramsey has actually approved the app.
