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

> See [`docs/architecture.md`](docs/architecture.md) for the system overview.
