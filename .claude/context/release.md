---
title: Release and CI
description: Tag-driven release flow, parallel CI jobs, and the reasoning behind e2e-only-on-release
---

# Release and CI

## Overview

Owns the tag-driven release flow, the parallel CI job graph, and the Chrome Web Store publish step. Files: `scripts/release.sh` and `package.json` scripts.

## Layout

- `.github/workflows/` owns the CI verify and release pipeline definitions

## Parallel jobs, gate only on data dependency

Static checks, unit tests, and build run in parallel on every PR. `needs` is reserved for jobs that require an artifact from a prior job or that are prohibitively expensive relative to their gate. `e2e-tests` gates on `build` because it requires the built extension. `release` gates on all three parallel jobs plus `e2e-tests` because it should not publish until everything passes.

## E2e tests run in release workflow only, not on PRs

Running Playwright on every PR adds 3–5 minutes per run and requires Chrome installation. The pre-push hook already runs `bun run check` locally. E2e is reserved for the release gate where correctness is critical before a publish.

## `changelogithub` generates all release notes, no CHANGELOG.md

`changelogithub` auto-generates release notes from conventional commits and writes them to the GitHub Release. There is no `CHANGELOG.md`. The GitHub Release is the changelog. This works because commits follow Conventional Commits and PRs are well-scoped, so the generated notes are the authoritative record.

## `npm version --no-git-tag-version` keeps version and tag in sync

`package.json` version is the source of truth for the zip filename (`crx-caret-{version}.zip`). The release script bumps the version with `npm version --no-git-tag-version`, then commits and tags manually using a conventional commit message. This keeps the commit message format consistent with the rest of the project rather than using npm's default format.

## Resumable releases

The release script detects an in-progress release branch and resumes from the last successful step. Branch push, tag push, and PR creation are each idempotent. Re-running after a flaky pre-push hook picks up where the previous run stopped.

## Verify workflow

Runs on every pull request to `main` and on manual dispatch. Three jobs run in parallel: static checks (typecheck, lint, format, spell check, shell check), unit tests with coverage, and a production build. A PR must pass all three before merging.

## Release workflow

Push a `v*` tag to trigger a release. Use the release script rather than tagging manually:

```bash
bun run release
```

The script prompts for a bump type (patch, minor, or major), updates `package.json`, commits with `chore(release): vX.X.X`, tags, and pushes. If a push fails (e.g. a flaky pre-push hook), re-run the same command. The workflow fires automatically on the tag push. Release notes are generated automatically by `changelogithub` from conventional commits and written to the GitHub Release.

### Job order

```plaintext
static-checks ─────────────┐
unit-tests     ─────────────┼─ release ─ publish
build          ─ e2e-tests ─┘
```

`static-checks`, `unit-tests`, and `build` run in parallel. `e2e-tests` gates on `build`. `release` gates on all three parallel jobs plus `e2e-tests`, creates the GitHub Release via `changelogithub`, and attaches the zip. `publish` gates on `release` and uploads to the Chrome Web Store. The publish step uses `continue-on-error` so a CWS rejection (e.g. pending review) does not fail the workflow.

The build produces `release/crx-caret-{version}.zip`, uploaded as a workflow artifact (7-day retention) and attached to the GitHub Release.

## Secrets

Add these to **Settings → Secrets and variables → Actions** before the first release.

- `CWS_EXTENSION_ID`: the extension ID in the Chrome Web Store URL, available after the first manual upload
- `CWS_CLIENT_ID`: from a Google Cloud OAuth 2.0 Desktop app client
- `CWS_CLIENT_SECRET`: from the same OAuth client
- `CWS_REFRESH_TOKEN`: obtained via the Google OAuth flow described below

### Google Cloud setup

1. Create a project at [Google Cloud Console](https://console.cloud.google.com).
2. Enable the Chrome Web Store API.
3. Go to **APIs & Services → Credentials → Create credentials → OAuth client ID**, set type to Desktop app. Copy the client ID and client secret.
4. Open the following URL in a browser, replacing `CLIENT_ID` with your value:
   `https://accounts.google.com/o/oauth2/auth?client_id=CLIENT_ID&redirect_uri=urn:ietf:wg:oauth:2.0:oob&scope=https://www.googleapis.com/auth/chromewebstore&response_type=code`
5. Sign in with the account that owns the developer dashboard. Copy the authorization code shown.
6. Exchange the code for a refresh token:
   ```bash
   curl -X POST https://oauth2.googleapis.com/token \
     -d client_id=CLIENT_ID \
     -d client_secret=CLIENT_SECRET \
     -d code=AUTHORIZATION_CODE \
     -d grant_type=authorization_code \
     -d redirect_uri=urn:ietf:wg:oauth:2.0:oob
   ```
   Copy the `refresh_token` value from the response.

## Chrome Web Store

For detailed steps on setting up the Chrome Web Store listing, including the initial manual publish and obtaining API credentials, refer to [store/listing.md](../../store/listing.md). After the initial setup, `bun run release` handles the full publish cycle.
