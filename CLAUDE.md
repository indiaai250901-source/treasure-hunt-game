# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A "Treasure Hunt" game: three treasure chests, one hides treasure (+$100), the other two hide a skeleton (-$50). Click a chest to open it; the game ends when the treasure is found or all three chests are opened. Built as a Figma Make / AI-generated React starter — most of `src/components/ui/` is an unmodified shadcn/ui component library pulled in wholesale, not hand-written for this app. There is no authentication; the game is a single self-contained page.

## Commands

- `npm install` — install dependencies
- `npm run dev` — start Vite dev server on `http://localhost:3000` (auto-opens browser)
- `npm run build` — production build, output to `build/` (note: not the Vite default `dist/`)

There is no test runner, linter, or type-check script configured in `package.json`. `tsc` is not a dependency, so don't assume `npm run build` performs type checking beyond what `@vitejs/plugin-react-swc` does (it strips types without checking them).

## Architecture

### Frontend (`src/`)
- **Entry point**: `src/main.tsx` mounts `App` from `src/App.tsx` into `#root`.
- **`src/App.tsx`** holds the entire game. It renders directly on mount — there is no auth gate, guest mode, router, or context layer to look for. Game state (`boxes`, `score`, `gameEnded`) is plain `useState`.
- **Box state model**: each box is `{ id, isOpen, hasTreasure }`; `initializeGame()` randomly assigns the treasure to one of the three boxes on mount and on "Play Again".
- **Animations** use `motion/react` (Motion, formerly Framer Motion) directly in JSX (`whileHover`, `whileTap`, `animate` on `rotateY`/`scale` for the chest flip).
- **Assets**: images in `src/assets/`, sound effects in `src/audios/` (`chest_open.mp3` for treasure, `chest_open_with_evil_laugh.mp3` for skeleton), imported directly as ES modules and referenced via Vite's asset pipeline. Sound effects are played by constructing `new Audio(importedMp3)` and calling `.play()`.
- **`src/components/ui/`** is the shadcn/ui-style primitive library (Radix UI wrappers + `class-variance-authority` variants). Treat it as vendored — extend by composing these primitives in `App.tsx` or new components rather than editing the primitives themselves, unless a real bug is found in one.
- **`src/components/figma/ImageWithFallback.tsx`** is a Figma Make platform helper for image fallbacks; also treat as vendored.
- **Styling**: Tailwind CSS v4, config-free (no `tailwind.config.js` — v4 uses CSS-based `@theme` config instead). `src/index.css` is Tailwind's compiled/expanded output committed as source (large, generated-looking — don't hand-edit utility classes there). `src/styles/globals.css` defines the actual design tokens: CSS custom properties for colors (`--background`, `--primary`, `--sidebar-*`, etc.) with a `.dark` class override block, plus `@custom-variant dark`. When theming, add/change tokens in `globals.css`, not `index.css`.
- **Path alias**: `@/` resolves to `src/` (configured in `vite.config.ts`). Note `vite.config.ts` also aliases every versioned import specifier (e.g. `sonner@2.0.3` → `sonner`) to match how the Figma Make export pins dependency versions in import statements — keep this pattern in mind if you see version-suffixed imports anywhere.
- **`cn()`** in `src/components/ui/utils.ts` (clsx + tailwind-merge) is the standard way to compose conditional Tailwind classes; used throughout `ui/`.

### No backend
There is no server in this repo. An Express + SQLite username/password auth layer (`server/`) was added at one point and has since been removed to restore the original single-page game; there is no session/auth state and no API to reach for score persistence or anything else. Score is purely in-memory React state, reset on refresh or "Play Again." If asked to add persistence or accounts again, that's new infrastructure, not wiring up something that already exists.

## Conventions seen in this repo

- Function components with inline Tailwind utility classes; no CSS modules or styled-components.
- `README.md` in this repo is a personal walkthrough log of Claude Code commands/workflows used while building this project (including prompts that were tried, such as an auth layer that was later removed), not user-facing documentation or a spec — it may record intent that isn't reflected in the current code.
