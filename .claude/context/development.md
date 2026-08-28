---
title: Development
description: Local setup, available bun scripts, project structure, and environment variables.
---

# Development

## Overview

Owns the local dev workflow: environment setup, the bun script surface, and the environment variables GitHub sync needs during development. Domain code lives in `src/`. Each domain's own context entry documents its layout, so this entry indexes how to run, build, and test the project rather than what each entry point does.

## Requirements

- [Bun](https://bun.sh) installed globally

## Setup

```bash
bun install
```

## Running locally

```bash
bun run dev
```

To load the extension in Chrome:

1. Open `chrome://extensions/`.
2. Enable "Developer mode".
3. Click "Load unpacked" and select the `dist` directory.

## Build

```bash
bun run build
```

Generates a production build and a zip in `release/`.

## Commands

- `bun run dev`: start development server with HMR
- `bun run build`: build for production
- `bun run build:dev`: build for development (includes dev env vars, e.g. GitHub seeding)
- `bun run format`: format with Prettier
- `bun run lint`: lint with ESLint
- `bun run lint:fix`: lint with auto-fix
- `bun run typecheck`: type check without building
- `bun run check:spell`: spellcheck
- `bun run test`: unit tests in watch mode
- `bun run test:run`: unit tests, single run
- `bun run test:ui`: unit tests with Vitest browser UI
- `bun run test:coverage`: unit tests with coverage report
- `bun run test:e2e`: end-to-end tests with Playwright
- `bun run test:e2e:ui`: end-to-end tests with Playwright UI
- `bun run test:e2e:report`: show last Playwright report
- `bun run check`: format, typecheck, lint, spellcheck, unit tests, and build verification
- `bun run check:full`: `check` plus end-to-end tests
- `bun run snapshot`: generate project snapshot in `.claude/.tmp/SNAPSHOT.md`
- `bun run screenshot`: generate extension screenshots
- `bun run release`: bump version and changelog, tag, and push to trigger the release workflow
- `bun run clean`: remove `node_modules` and temporary files
- `bun run update`: update dependencies and run full verification

## Environment variables

Set these environment variables in `.env` or your shell for local development, particularly for GitHub sync:

- `VITE_GITHUB_PAT`: Your GitHub Personal Access Token.
- `VITE_GITHUB_OWNER`: The owner of the GitHub repository (e.g., `my-org`).
- `VITE_GITHUB_REPO`: The name of the GitHub repository (e.g., `my-prompts`).
- `VITE_GITHUB_BRANCH`: The branch to sync from (defaults to `main`).
- `VITE_GITHUB_SNIPPETS_PATH`: The path within the repository where snippets are stored (defaults to `snippets`).

## Commit standards

This project follows [Conventional Commits](https://www.conventionalcommits.org/). The `aitk:git-commit` skill generates the message and carries the format rules.
