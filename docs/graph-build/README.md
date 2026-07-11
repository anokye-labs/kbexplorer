# Graph build

How a kbx knowledge base is **built** — the path from a repository's content to a
pure graph and, finally, to something a person or an agent can read. Where
[workflows](../workflows/) cover the *operational* "how" of running a dataset,
this set opens the box and traces the *internal* pipeline: one repository, two
possible sources, one engine, four interchangeable render targets.

These pages are the narrative companion to
[Layer 3 — Engine](../architecture.md#layer-3--engine) in the architecture doc.
Where architecture states the contract, these pages walk it end to end, each
fronted by an animated diagram of the stage it describes.

| Document | What it covers |
|----------|---------------|
| [Baked vs live](baked-vs-live.md) | The one pipeline fed by two sources. `VITE_KB_LOCAL=true` selects a build-time-baked manifest (zero API calls); the default selects the live GitHub API. Both converge on `loadKnowledgeBase(source, config)`. |
| [Sources → RepoData](sources-to-repodata.md) | The `Source` contract (`retrieve` / `get`), `getRepoData()` → `RepoData`, affordances advertised **per retrieval**, and the git ≠ GitHub composite. |
| [Providers → graph](providers-to-graph.md) | `registerProviders`, `orchestrateWithTransforms` (collect → transform → cluster → build), `DEFAULT_TRANSFORMS`, and how `buildGraph` derives the deduped, connected `KBGraph`. |
| [Graph → representation](graph-to-representation.md) | The pure `KBGraph` → `RepresentationRegistry` → four targets (`spa`, `json-ld`, `llm-context`, `copilot`), and the purity guarantee that keeps the data targets engine-free. |

## How to read the diagrams

Each page opens with a hand-authored, animated SVG in the same visual language as
the [workflows](../workflows/) diagrams: three horizontal bands, with numbered
steps flowing left → right.

- **People · surfaces** (teal, top) — what humans and agents finally consume:
  the rendered SPA, or the `json-ld` / `llm-context` / `copilot` targets.
- **The overlay · kbx** (violet, middle) — what kbx itself does: the `Source`,
  `getRepoData`, the providers, the orchestrator, `buildGraph`, and the
  representation registry.
- **Host · systems of record** (slate, bottom) — where the bytes actually live:
  `config.yaml` and `content/*.md` in git, the live GitHub API, and the
  build-time-baked `repo-manifest.json`.

The band mapping is deliberately the *same* as the workflows set, so the two
reading experiences compose: inputs sit in the host band, everything kbx does
sits in the overlay, and the surfaces people touch sit on top. Steps pop in and
edges draw in sequence, then the diagram holds, fades, and rebuilds — a build-up
idiom, because a graph is *assembled*, not handed over whole. Animation is pure
CSS and honors `prefers-reduced-motion`.

## Reading order

Read them in pipeline order:

1. [Baked vs live](baked-vs-live.md) — the two ways bytes enter, and where they
   converge. Start here for the big picture.
2. [Sources → RepoData](sources-to-repodata.md) — how a source turns a system of
   record into one normalized `RepoData` bundle.
3. [Providers → graph](providers-to-graph.md) — how providers, transforms, and
   `buildGraph` derive the pure graph.
4. [Graph → representation](graph-to-representation.md) — how that one graph
   becomes a website, JSON-LD, an LLM context, or a canvas.

## Grounding

Every code fact on these pages is grounded against
[`kbexplorer-template`](https://github.com/anokye-labs/kbexplorer-template) at
tag **`v0.4.1`** — the ref the
[showcase pins](../../.github/workflows/showcase.yml) (`TEMPLATE_REF`). Template
file links point at that tag so the referenced lines match the prose. Per the
repo's convention, type-level contracts link to their canonical definitions in
[`kbexplorer-core`](https://github.com/anokye-labs/kbexplorer-core) rather than
being re-described here.

## Related documentation

- [Architecture](../architecture.md) — the four-layer model
  (Sources → Providers → Engine → Representation) these pages walk through.
  In particular [Layer 3 — Engine](../architecture.md#layer-3--engine).
- [Surfaces](../surfaces.md) — the two render surfaces over the same graph: the
  human-facing SPA and the agent-facing embeddable Copilot canvas.
- [Workflows](../workflows/) — the operational companion: creating, hosting,
  updating, and governing a dataset day to day.
