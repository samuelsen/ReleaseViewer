# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Single-file static web app (`index.html`) — no build step, no dependencies, no package manager. Open directly in a browser.

**Purpose:** Monitor GitHub Actions deployment statuses across repos and environments. Tracks per-environment status for a set of user-configured pipelines.

## Running / testing

```bash
# Serve locally (any static server works):
python3 -m http.server 8080
# then open http://localhost:8080
```

For UI testing without a token or network, use **demo mode** ("Try demo with sample data" on the auth screen, or "Load sample data" from the dashboard menu). It injects sample repos with envs `utv`/`staging`/`preprod`/`prod` and randomized statuses.

## Architecture

Everything lives in `index.html`. Three logical layers, top-to-bottom in the `<script>` block:

1. **Persistence (`Config`)** — getter/setter object backed by `localStorage`. Keys: `rv_token`, `rv_repos`, `rv_user`, `rv_base_url`, `rv_view_mode`, `rv_theme`, `rv_hide_env_only`, `rv_favorites`, `rv_demo`. Repos stored as JSON array of repo-config objects (`owner`, `repo`, `workflowId`, `envType`, `environments`). Demo repos additionally carry `demo:true` + embedded `_statuses`/`_ts`.

2. **GitHub API (`GH`)** — thin fetch wrapper. Supports both github.com and GitHub Enterprise (GHE) via `Config.baseUrl`. All calls go through `GH.fetch(path, opts)` which injects auth headers. GHE omits the `X-GitHub-Api-Version` header.

3. **UI (render functions)** — no framework. Imperative DOM manipulation. Entry point is `render()` which switches on `Config.token || Config.demo` (auth screen vs shell) and `State.view` (`'dashboard'` | `'detail'`). Each view is a standalone `render*()` function that returns a DOM node. Re-renders are full replacements via `$app.innerHTML = ''` + `$app.append(...)`.

**State:** Two globals — `Config` (persisted) and `State` (in-memory: `view`, `detailRepo`, `modal`).

**Two environment detection modes** (`envType`):
- `github_environments` — uses the GitHub Environments API + Deployments API. Preferred.
- `job_names` — fallback; scans workflow run jobs by name regex for deploy-related names.

**Demo mode** — `loadDemoData()` seeds sample repos; `makeDemoRepo(owner, repo)` builds one. `fetchEnvStatuses`/`fetchLatestTs` short-circuit on `rc.demo` (return embedded data, no API). `renderDemoRuns(rc, el)` fakes the detail-page run table. `renderAddRepoModal` branches to a simple owner/repo form when `Config.demo`.

**`GH` API methods:** `getUser`, `getEnvironments`, `getWorkflowRuns`, `getWorkflows`, `getRunJobs`, `getDeployments`, `getDeploymentStatuses`, `listUserRepos`, `listUserOrgs`, `listOrgRepos`. All defined on the `GH` object; add new API calls there.

**Render tree:**
- `render()` → `renderAuth()` or `renderShell()`
- `renderShell()` → `renderHeader()` + `renderDashboard()` or `renderDetail(rc)`
- `renderDashboard()` → `renderRepoGroup()` → `renderRepoCard()` → `renderEnvItem()`
- Modal overlay: `State.modal = 'add-repo'` → `renderAddRepoModal()` appended to shell

**Helper functions:** `fetchEnvStatuses(rc)` fetches live status per environment; `fetchLatestTs(rc)` fetches latest run timestamp for a card. `detectEnvironments(owner, repo, workflowId)` auto-detects envs during repo setup.

## Key conventions

- All HTML is built as template literals inside `innerHTML` or via `document.createElement`. No external templating.
- User-supplied strings always pass through `esc()` before insertion into HTML.
- Status normalization happens in `statusCls()` and `statusLabel()` — add new GitHub states there.
- Theme is CSS custom properties with `[data-theme]` attribute toggling; system default via `prefers-color-scheme`.
- No linter or formatter configured. Match surrounding code style.
