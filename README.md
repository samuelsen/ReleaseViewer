# Release Viewer

A single-file static web app for monitoring **GitHub Actions deployment statuses** across multiple repositories and environments. No build step, no dependencies, no backend — just open `index.html`.

## Features

- Track per-environment deployment status (e.g. `utv`, `staging`, `preprod`, `prod`) across many repos
- Two views: **per-pipeline** cards or **grouped-by-repo**
- Mark **favorites** and filter to show only favorites
- Search, sort (A–Z / latest run), light/dark/system themes
- **GitHub Enterprise (GHE)** support alongside github.com
- **Demo mode** — explore the UI with sample data, no token required
- Detail view per repo: current deployments + recent workflow runs

## Quick start

Open `index.html` directly in a browser, or serve it with any static file server.

### Connecting to GitHub

On the auth screen, paste a **Personal Access Token** with the required scopes:

- `repo` (private repos) **or** `public_repo` (public only)
- `read:org` (for organization repos)

For GitHub Enterprise, also enter your GHE hostname (e.g. `github.mycompany.com`).

The token is stored in your browser's `localStorage` and never leaves the page except in direct API calls to GitHub.

### Demo mode

Click **"Try demo with sample data"** on the auth screen (or **Load sample data** from the dashboard menu) to explore the app with generated sample repos — no token or network needed. In demo mode you can add repos, browse detail pages, and toggle favorites.

## Environment detection modes

Each tracked pipeline uses one of two `envType` strategies:

- **`github_environments`** (preferred) — reads the GitHub Environments + Deployments APIs.
- **`job_names`** (fallback) — scans workflow run jobs, matching deploy-related job names by regex.

## Architecture

Everything lives in `index.html`, organized top-to-bottom in the `<script>` block:

1. **Persistence (`Config`)** — getter/setter object backed by `localStorage`. Keys include `rv_token`, `rv_repos`, `rv_user`, `rv_base_url`, `rv_view_mode`, `rv_theme`, `rv_hide_env_only`, `rv_favorites`, `rv_demo`.
2. **GitHub API (`GH`)** — thin `fetch` wrapper; injects auth headers and supports github.com + GHE.
3. **UI** — no framework. Imperative DOM building. `render()` switches between the auth screen and the app shell; each view is a standalone `render*()` function.

State lives in two globals: `Config` (persisted) and `State` (in-memory: `view`, `detailRepo`, `modal`).

## Notes

- All user-supplied strings pass through `esc()` before insertion into HTML.
- Status normalization happens in `statusCls()` / `statusLabel()`.
- No linter or formatter is configured — match the surrounding code style.
</content>
</invoke>
