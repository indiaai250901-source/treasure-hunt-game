---
description: Build and deploy the treasure hunt game to GitHub Pages
argument-hint: "[owner/repo]"
allowed-tools: Bash(git:*), Bash(npm run build:*), Bash(npx gh-pages:*), Bash(npx:*), Bash(gh:*), Read, Edit
---

Deploy this static Vite app to GitHub Pages via the `gh-pages` branch.

Arguments: `$ARGUMENTS` — the target GitHub repo as `owner/repo` (e.g. `octocat/treasure-hunt`). Required the first time this command runs in a repo with no `origin` remote configured yet.

Steps:
1. Run `git rev-parse --is-inside-work-tree` and `git remote -v` to check current state.
   - If this directory is not yet a git repo, run `git init`, then stage and commit the existing source with the user's confirmation before doing anything else.
   - If there's no `origin` remote:
     - If `$ARGUMENTS` is empty, stop and ask the user for the target repo (`owner/repo`).
     - This command does not create GitHub repos. Confirm with the user that a repo named `$ARGUMENTS` already exists on GitHub (an empty repo is fine). If the `gh` CLI is installed and authenticated, offer to create it instead with `gh repo create $ARGUMENTS --public --source=. --remote=origin`.
     - Otherwise add the remote manually: `git remote add origin https://github.com/$ARGUMENTS.git`.
2. Work out the repo name (from `$ARGUMENTS`, or parsed out of the existing `origin` URL). GitHub Pages project sites are served at `https://<owner>.github.io/<repo>/`, not the domain root, so the build needs `base: '/<repo>/'`.
   - Check `vite.config.ts`'s `defineConfig({...})` for a `base` key. If it's missing or doesn't match `/<repo>/`, add/update it (e.g. `base: '/<repo>/',` alongside the existing `build:` key). Confirm with the user before overwriting a `base` that's already set to something else.
3. If there are uncommitted or unpushed changes on the default branch, commit/push them first (ask the user before the very first push to a brand-new remote — this is visible to others).
4. Run `npm run build` (output goes to `build/` per `vite.config.ts` — not the Vite default `dist/`). Stop and report the error if it fails; do not deploy a broken build.
5. Confirm `build/index.html` exists after the build.
6. Publish the `build/` folder to the `gh-pages` branch: `npx gh-pages -d build -m "Deploy $(git rev-parse --short HEAD)"`. This creates the `gh-pages` branch automatically on first run.
7. First deploy only — GitHub Pages needs to be pointed at the `gh-pages` branch once:
   - If `gh` is available and authenticated, check `gh api repos/$ARGUMENTS/pages` and, if unset, enable it with `gh api repos/$ARGUMENTS/pages -X POST -f "source[branch]=gh-pages" -f "source[path]=/"`.
   - Otherwise tell the user to do it manually: repo → Settings → Pages → Source → "Deploy from a branch" → `gh-pages` / `/(root)`.
8. Report the live URL back to the user: `https://<owner>.github.io/<repo>/`. Note that GitHub Pages can take 1-2 minutes after the first push to actually go live.

Do not force-push, delete branches, or change repo visibility without explicit user confirmation.
