---
description: Build and deploy the treasure hunt game to Vercel
argument-hint: "[preview]"
allowed-tools: Bash(npm run build:*), Bash(npx vercel:*), Bash(ls:*)
---

Deploy this static Vite app to Vercel.

Arguments: `$ARGUMENTS` — if it contains "preview", do a preview deploy; otherwise deploy to production.

Steps:
1. Run `npm run build` (production build, output goes to `build/` per `vite.config.ts` — NOT the Vite default `dist/`). If it fails, stop and report the error — do not deploy a broken build.
2. Confirm `build/index.html` exists after the build.
3. Deploy with the Vercel CLI via `npx`:
   - Production (default, no "preview" in `$ARGUMENTS`): `npx vercel --prod`
   - Preview (if `$ARGUMENTS` contains "preview"): `npx vercel`
   - `vercel.json` in the repo root already pins `outputDirectory` to `build` and `buildCommand` to `npm run build`, so the CLI/dashboard build settings don't need manual overrides.
   - If this is the first deploy and the CLI prompts for project setup/login, surface those prompts to the user rather than guessing answers.
4. Report back the deployment URL Vercel prints, and whether it was a production or preview deploy.

Do not push to git or modify branches as part of this command — this only builds and deploys via the Vercel CLI.
