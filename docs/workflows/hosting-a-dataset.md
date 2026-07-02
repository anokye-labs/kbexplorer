# Hosting a dataset

Deploying a built kbx knowledge base so others can use it. Three deployment
patterns exist today: GitHub Pages (the showcase pattern), Azure App Service
(a real-world example), and a localhost search service.

For _creating_ the dataset in the first place, see
[creating a dataset](creating-a-dataset.md).

## What `kbx build` produces

```bash
npx kbx build    # -> dist/kb/
```

`kbx build` runs the template's Vite production build in local mode
(`VITE_KB_LOCAL=true`). The output is a self-contained static bundle in
`dist/kb/` with a baked-in manifest — no runtime GitHub token or API calls
needed. The manifest is generated from local `content/`, the file tree, the
README, and best-effort `gh` issues/PRs/commits.

## Pattern 1 — GitHub Pages (the showcase)

This repo itself is the reference example: the
[live showcase](https://anokye-labs.github.io/kbexplorer/) deploys the
`kbexplorer-template` SPA to GitHub Pages from this docs repo.

### How it works

The
[`showcase.yml`](https://github.com/anokye-labs/kbexplorer/blob/main/.github/workflows/showcase.yml)
workflow:

1. Checks out the template at a **pinned release tag** (currently `v0.4.0`,
   set via the `TEMPLATE_REF` env var in the workflow).
2. Checks out this (host) repo as the content source.
3. Runs `node scripts/generate-manifest.js` with `VITE_KB_LOCAL=true` and
   `VITE_KB_HOST_ROOT` pointing at the host checkout — baking config,
   authored content, file tree, README, and (best-effort) issues/PRs into
   `src/generated/repo-manifest.json`.
4. Runs `npx vite build` with `VITE_BASE_PATH=/kbexplorer/`.
5. Copies `index.html` to `404.html` for SPA routing.
6. Uploads and deploys via `actions/upload-pages-artifact` +
   `actions/deploy-pages`.

### Triggers

- `push` to `main` (but note: agent auto-merges via `GITHUB_TOKEN` do not emit
  a `push` event — those deploys come from the backstops below).
- `workflow_dispatch` (manual).
- `schedule: cron '17 4 * * *'` — daily rebuild keeps baked repo-aware content
  (issues/PRs) reasonably fresh.

### Not a required status check

`showcase.yml` never gates PRs or auto-merge. It deploys _after_ merge, on
demand, or on the daily schedule.

### Adopting this pattern for your own repo

1. Copy `showcase.yml` into your repo's `.github/workflows/`.
2. Set `TEMPLATE_REPO` and `TEMPLATE_REF` to the template and version you want.
3. Ensure your repo has authored content in `content/` or is repo-aware.
4. Enable GitHub Pages (Settings -> Pages -> Source: GitHub Actions).

The template advises host repos to **pin to a release tag**, not `main`. Bump
the tag via a normal issue-first PR when a new template release ships.

## Pattern 2 — Azure App Service (real-world example)

The [Anokye System site](https://system.anokye.dev) is a production deployment
that uses Azure App Service with Express SSR, OAuth via a GitHub App, and OIDC
deploy auth. It is cited here as an example of a non-Pages deployment target,
not as kbx-specific infrastructure.

### Deployment mechanics

- **Build**: Astro SSR build producing `dist/server/entry.mjs` (handler) +
  `dist/client/` (static assets), served by an Express wrapper.
- **CI/CD**: `deploy-app-service.yml` authenticates to Azure via OIDC
  (`azure/login@v3`) + `azure/webapps-deploy@v3`, with a post-deploy
  smoke-test job hitting `/api/auth/me` and `/api/auth/login`.
- **Auth**: server-side, enforced by Express middleware — not client-side.
- **DNS**: custom domain with managed SSL certificate.

For provisioning details, see
[`anokye-system/site/.agent-context/azure-setup.md`](https://github.com/anokye-labs/anokye-system/blob/main/site/.agent-context/azure-setup.md)
and
[`deployment.md`](https://github.com/anokye-labs/anokye-system/blob/main/site/.agent-context/deployment.md).

### Relevance to kbx

Any environment that can serve static files (or an SSR build) can host a kbx
dataset. The key is `dist/kb/` (or the template's `dist/`) — serve those files
and set `VITE_BASE_PATH` appropriately. Azure App Service, Vercel, Netlify,
Cloudflare Pages, and a plain `nginx` or `caddy` config all work.

## Pattern 3 — Search service

The built explorer is static (self-contained local mode, no backend), so it
does not run search by default. To add semantic search:

```bash
# 1. Build the search index
kbx search-index                    # -> .search/ artifacts

# 2. Start the search service
kbx search-serve                    # localhost HTTP service
# or:
npx @anokye-labs/kbexplorer-search serve --dir .search --port 7700

# 3. Point the template at the service
VITE_SEARCH_SERVICE_URL=http://127.0.0.1:7700 npm run dev   # in kbexplorer-template
```

The search service exposes a small HTTP contract:

| Method & path | Body | Response |
|---------------|------|----------|
| `GET /health` | -- | `{ status, unitCount, model }` |
| `GET /stats` | -- | `{ unitCount, model, dimensions, contentHash, version }` |
| `POST /search` | `{ query, limit?, cluster?, entityType?, minScore?, graphRanking? }` | `{ results: SearchResult[], suggestions: RelatedSuggestion[] }` |

For details on how the search corpus is built, see
[search corpus updates](search-corpus-updates.md).

## Environment variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `VITE_KB_LOCAL` | `true` for self-contained local mode | Build time |
| `VITE_KB_HOST_ROOT` | Path to the host repo's root (for the showcase pattern) | Build time |
| `VITE_BASE_PATH` | URL base path (e.g. `/kbexplorer/`) | Build time |
| `GH_TOKEN` | GitHub token for manifest generation (issues/PRs) | Build time (best-effort) |
| `VITE_SEARCH_SERVICE_URL` | URL of the search service | Runtime (optional) |

## Next steps

- [Updating a dataset](updating-a-dataset.md) — the refresh loop after first
  deploy.
- [Governance with rulesets](governance-with-rulesets.md) — how CI and branch
  protection manage the dataset repo.

---

> The showcase is a `spa` representation target rendering the pure `KBGraph`;
> see [architecture.md](../architecture.md) for the four-layer model and the
> [surfaces](../surfaces.md) doc for how the SPA and the embeddable Copilot
> canvas are different representation targets over the same graph.
