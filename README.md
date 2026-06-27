# kbexplorer

Official **documentation** and a **live showcase** for the kbexplorer system.

This repo holds **no application code**. The code lives in:

- [`kbexplorer-core`](https://github.com/anokye-labs/kbexplorer-core) — shared
  contracts (types, `kg://` identity, relation taxonomy, JSON-LD helpers, and the
  Source / Provider / Representation interfaces).
- [`kbexplorer-cli`](https://github.com/anokye-labs/kbexplorer-cli) — the CLI that
  derives content and drives the explorer.
- [`kbexplorer-template`](https://github.com/anokye-labs/kbexplorer-template) — the
  SPA that renders a knowledge graph in the browser.

## What's here

- `docs/` — the architecture documentation:
  - the four layers — **Sources → Providers → Engine → Representation**,
  - the source / affordance model (affordances are advertised **per retrieval**,
    not per resource type, and resources carry hypermedia `links`),
  - the provider and representation **extension points** (bring your own local or
    third-party providers and render targets).
- a deployed **showcase** of kbexplorer (added by a follow-up task).

> See [`docs/architecture.md`](docs/architecture.md) for the system overview.
