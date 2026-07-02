# kbexplorer

Official **documentation** and a **live showcase** for the kbexplorer system.

This repo holds **no application code**. The code lives in:

- [`kbexplorer-core`](https://github.com/anokye-labs/kbexplorer-core) — shared
  contracts (types, `kg://` identity, relation taxonomy, JSON-LD helpers, and the
  Source / Provider / Representation interfaces).
- [`kbexplorer-cli`](https://github.com/anokye-labs/kbexplorer-cli) — the CLI that
  derives content, drives the explorer, and serves the embeddable Copilot canvas.
- [`kbexplorer-template`](https://github.com/anokye-labs/kbexplorer-template) — the
  SPA that renders a knowledge graph in the browser (and, additively, the
  embeddable canvas entry).
- [`kbexplorer-search`](https://github.com/anokye-labs/kbexplorer-search) — the
  semantic-search companion module, driven through the CLI.
- [`kbexplorer-provider-rich-markdown`](https://github.com/anokye-labs/kbexplorer-provider-rich-markdown) —
  the loadable provider that ingests a rich-Markdown document into a graph
  fragment.

## What's here

- [`docs/architecture.md`](docs/architecture.md) — the architecture documentation:
  - the four layers — **Sources → Providers → Engine → Representation**,
  - the source / affordance model (affordances are advertised **per retrieval**,
    not per resource type, and resources carry hypermedia `links`),
  - the **do-seam** — the protocol-neutral affordance action layer and its
    three interchangeable delivery adapters (extension-tool, MCP, canvas),
  - the provider and representation **extension points** (bring your own local or
    third-party providers and render targets).
- [`docs/surfaces.md`](docs/surfaces.md) — the **two render surfaces** over the
  same graph: the human-facing SPA showcase (GitHub Pages) and the
  agent-facing embeddable Copilot canvas (served by `kbexplorer-cli` over a
  loopback server). They are different Representation targets, not two
  versions of one product.
- [`docs/history.md`](docs/history.md) — a verified, dated changelog across all
  five repos, plus an honest "what's actually shipped vs. designed" status.
- [`docs/decisions/`](docs/decisions/) — design decisions worth preserving:
  [connect-not-merge for cross-source data](docs/decisions/conflation-correction.md)
  (decided, shipped) and an
  [open question on rich-Markdown-by-default](docs/decisions/markdown-rendering-default.md)
  (explicitly **not** decided).
- [`docs/canvas-dev-loop.md`](docs/canvas-dev-loop.md) — a verified, run-it-yourself
  guide for iterating on the embeddable canvas locally.
- [`docs/adoption-paved-path.md`](docs/adoption-paved-path.md) — a holistic
  adoption roadmap for making kbx integration into another repository a guided,
  diagnosable path.
- a deployed **showcase** of kbexplorer — the `spa` representation rendering a
  sample knowledge base in the browser.

## The kbx story

Beyond the architecture, this repo tells the **full kbx story** so a reader — or an
agent — can understand who kbx is for and how it gets built:

- [`docs/personas.md`](docs/personas.md) — the people who use kbx.
- [`docs/journeys.md`](docs/journeys.md) — end-to-end persona journeys.
- [`docs/user-stories.md`](docs/user-stories.md) — the user-story **demand map**
  (families A–K), each story linked to the issue that delivers it.
- [`docs/roadmap.md`](docs/roadmap.md) — the two programs (foundation + plugin) and the
  cross-repo, wave-by-wave execution plan.

Together these close the loop: start from a person's need and walk to the code that
satisfies it, or start from an epic and find who it is for. The work is tracked under
[#23](https://github.com/anokye-labs/kbexplorer/issues/23).

## Live showcase

**<https://anokye-labs.github.io/kbexplorer/>** — the
[`kbexplorer-template`](https://github.com/anokye-labs/kbexplorer-template) SPA,
built in self-contained local mode over a sample knowledge base and published to
GitHub Pages from this repo.

The deployed showcase is static (self-contained local mode, no backend), so it
doesn't run a search service by default. To try semantic search over this
repo's own content: `kbx search-index` to build `.search/` artifacts, then
`kbx search-serve` (or `npx @anokye-labs/kbexplorer-search serve`) and point a
local template dev build's `VITE_SEARCH_SERVICE_URL` at it — see
[Search in `docs/architecture.md`](docs/architecture.md#search--a-first-class-representation-over-the-graph).

> See [`docs/architecture.md`](docs/architecture.md) for the system overview.
