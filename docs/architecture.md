# kbexplorer architecture

> Scaffold. The full treatment lands via the "Author architecture docs" task
> (anokye-labs/kbexplorer#2). This file fixes the vocabulary and the layer
> boundaries so the rest of the docs can build on them.

kbexplorer is organized as four decoupled layers. Data flows left to right; each
layer depends only on the **contracts** in
[`kbexplorer-core`](https://github.com/anokye-labs/kbexplorer-core), never on a
concrete implementation of its neighbor.

```
Sources  ─▶  Providers  ─▶  Engine  ─▶  Representation
(SoR adapters)            (pure KBGraph)  (json-ld / llm-context / spa)
```

## 1. Sources (systems of record)

A **Source** adapts a system of record (the GitHub API, a local git worktree, a
static manifest file, a third-party service) into retrievable **resources**.

Affordances are advertised **per retrieval**, not per resource type or instance:

- A Source declares a *possible universe* of affordances (advisory).
- Each **retrieved resource** carries the affordances actually allowed *right now*
  (`read` / `write` / `stage`) plus hypermedia `links`. The same resource can come
  back with different affordances on a later retrieval (auth, branch protection,
  lock state, worktree, rate limits).

The **staging area** is a first-class, retrievable resource: a staged file MUST
carry `links: [{ rel: 'staging-area', href }]`.

**Git ≠ GitHub.** A `GitHubApiSource` is a composite: Git resources
(`file` / `tree` / `commit` / `staging-area`, with `read` / `write` / `stage`) and
GitHub resources (`issue` / `pull-request` / `release`, with `comment` / `close` /
`merge` / `convert-to-draft`). `draft` / `proposed` / `PR` are GitHub concepts —
never git `stage` sub-states.

## 2. Providers

A **GraphProvider** turns resources from one or more Sources into graph fragments
(nodes + edges). Providers are pluggable: **local ES modules first**, npm packages
second. A provider declares the affordances it needs; the engine fails fast if a
retrieval can't satisfy them.

## 3. Engine

The engine resolves providers and emits a **pure `KBGraph`** — data only, no
styling, no rendering, no I/O. This is the stable artifact everything else
consumes.

## 4. Representation

A **Representation** renders the pure graph for a target. Interchangeable targets:

- `json-ld` — deterministic linked-data serialization.
- `llm-context` — **neighbor-anchored** token-budgeted context for an LLM: anchored
  on context node(s), emits nearest neighbors ranked by edge weight to a budget,
  and emits navigable `kg://` links for relevant-but-unexpanded neighbors. It never
  returns the whole graph.
- `spa` — the browser explorer (`kbexplorer-template`). See the
  **[live showcase](https://anokye-labs.github.io/kbexplorer/)**.

## Extension points

- **Bring your own Source** — adapt a new system of record.
- **Bring your own Provider** — local module or published package.
- **Bring your own Representation** — a new render target off the same pure graph.
