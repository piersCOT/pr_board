# PR Activity Board

A static dashboard for visualising pull request activity from Azure DevOps. No backend, no build step — just a single HTML file served via GitHub Pages.

**Live:** [pierscot.github.io/pr_board](https://pierscot.github.io/pr_board/)

## Features

- Searchable project and repo selectors in the header breadcrumb
- Bar chart of PRs created per day, auto-grouped by week for long ranges
- Date range presets (7d, 14d, 30d, 90d, This month, Last month, custom)
- Add users from the org directory; click cards to filter chart and table
- "Show all PRs in repo" — adds everyone with PRs in the current filter, removes those with none
- Per-PR file and line stats fetched in the background; aggregated per user card
- Download a self-contained static HTML snapshot — no PAT or ADO calls required; user tiles still filterable
- Three themes (Green, Light, Dark); all filter state and data cached in `localStorage`

## Getting started

1. Open the [live page](https://pierscot.github.io/pr_board/) (or serve `index.html` locally)
2. Paste an Azure DevOps Personal Access Token (hover the **?** icon for instructions)
3. Add users from the dropdown
4. Data loads automatically

### Creating a PAT

1. Go to [dev.azure.com](https://dev.azure.com/capitalontap/_usersSettings/tokens) > User Settings > **Personal Access Tokens**
2. Click **+ New Token**
3. Set scopes to **Custom defined**, then enable:
   - **Code** > Read
   - **Identity** > Read
   - **Graph** > Read *(optional — enables full org user directory in the dropdown)*
4. Copy the token and paste it into the app

### Running locally

```bash
npx serve -l 9090 .
# then open the URL printed in the terminal
```

## Security

This dashboard is safe to host publicly. Here is a full analysis:

### No secrets in source

- The PAT is never hardcoded — it is entered by each user and stored in their browser's `localStorage` only
- `.env` and `.claude/` are in `.gitignore` and are never committed
- No API calls are made without a valid PAT; every fetch function requires it as a parameter
- All `useEffect` hooks that trigger network requests guard with `if (!pat.trim()) return`

### No PII in source

- `DEFAULT_USERS` is an empty array — no names, emails, or identifiers are in the codebase
- User data only appears after a visitor authenticates with their own PAT
- Previously added users are persisted in the visitor's own `localStorage`, scoped to the page origin

### What is visible in the source

| Item | Risk | Notes |
|------|------|-------|
| Org name (`capitalontap`) | None | Public-facing identifier, not exploitable |
| Default project name (`CapitalOnTapTests`) | None | Requires authentication to access; users can switch projects via dropdown |
| Azure DevOps API endpoint patterns | None | Publicly documented APIs |
| PAT generation link | None | Only works if the visitor is already authenticated to the org |

### What a visitor without a PAT sees

A static page with a title, an empty PAT input, date range buttons, and an empty users panel. Zero network requests are made. No data is exposed.

### Third-party dependencies

All loaded from `unpkg.com` CDN (standard, widely-used):

- React 18
- ReactDOM 18
- PropTypes 15
- Recharts 2.12.7
- Babel Standalone (for JSX transpilation)
- JetBrains Mono font (Google Fonts)

No SRI integrity hashes are set on CDN scripts. This is a minor hardening opportunity but not a practical vulnerability for this use case.

### Data flow

```
Browser localStorage (PAT) --> Azure DevOps REST API --> Browser (render)
```

The PAT is sent only to `dev.azure.com`, `vssps.dev.azure.com`, and `vsaex.dev.azure.com` via HTTPS Basic auth headers. It is never sent to any other server, logged, or included in URLs.

