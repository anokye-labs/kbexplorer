# Structured-file visualization — provider-shipped lenses and representations

> **Design doc for [Epic anokye-labs/kbexplorer#130](https://github.com/anokye-labs/kbexplorer/issues/130).**
> This is the architecture treatment; the canonical contracts it describes live
> in [`kbexplorer-core`](https://github.com/anokye-labs/kbexplorer-core) and the
> reference implementation in
> [`kbexplorer-template`](https://github.com/anokye-labs/kbexplorer-template).
> Where this doc names a *type*, it links to the core contract rather than
> restating it — the contract is the source of truth, and this prose will go
> stale the moment the contract moves. Verify against the linked source before
> relying on any shape here.

## The outcome

Every file KB Explorer surfaces should resolve to a **real, typed
visualization** — not a raw-text fallback — and the *same* structured resource
should be viewable through **multiple named lenses**, chosen by configuration or
annotation rather than by forking code. An `.ics` schedule becomes a **month
calendar**, an **agenda**, *or* a **Gantt**; a Graphviz DOT or JSON-Canvas graph
renders as a live graph, an animated flow, or a board — the underlying node data
never changes, only the lens over it does. This is the still-unfiled
"lenses / multi-lens-by-annotation" clause of near-term outcome **O3** in the
[requirements](https://github.com/anokye-labs/kbexplorer/issues/12), sequenced
under the [program roadmap](roadmap.md).

Two further properties make it an *extension story*, not a hard-coded feature
list:

1. **Providers ship both halves.** A loadable provider package can bring a new
   type's **ingestion** (data) *and* its **viewers / block renderers** (render)
   — including new lenses for resources some *other* provider ingested (bespoke
   GitHub issue / PR / Wikipedia views over nodes the generic provider brought
   in). `npm install` + one `config.yaml` entry, no core or template edit.
2. **Embedded blocks render live through the same components.** A fenced
   ` ```dot `, ` ```ics `, or ` ```canvas ` block inside a Markdown document
   renders through the *same* lens component as the equivalent whole-file node,
   replacing today's pre-built-SVG-only fallback.

## Where this sits in the architecture

This epic is almost entirely about the **render half** of
[Layer 4 — Representation](architecture.md#layer-4--representation). The pure
[`KBGraph`](https://github.com/anokye-labs/kbexplorer-core/blob/main/src/graph.ts)
is unchanged as *data*; what changes is how a node's `type` (and now its
*lenses*) resolve to a concrete viewer inside the `spa` and `copilot`
representation targets ([the two surfaces](surfaces.md)), and how a loadable
provider is allowed to *contribute* those viewers.

The [one-way dependency rule](architecture.md#the-one-way-dependency-rule) is the
constraint that shapes the whole design:

- **Core stays framework-free.** It never imports React or a DOM. So core can
  carry a *specifier* for a render module and *pure-data* view-models, but never
  a component. See [Tier 0](#tier-0--canonical-view-models-stack-free) below.
- **The engine stays render-free** (per
  [template#463](https://github.com/anokye-labs/kbexplorer-template/issues/463) /
  [#467](https://github.com/anokye-labs/kbexplorer-template/issues/467)):
  the CLI's composite ingest and the engine run in Node with no DOM. They must
  load a render-contributing provider's *data* half without ever evaluating its
  React module graph.
- **Components live template-side**, bound through opaque host hooks.

That last point forces a **module-graph isolation** rule that runs through every
piece of this epic: importing a provider's `.` (data) entry must *never* pull in
its render module graph. The render half is declared as a *separate* package
subpath the host only imports when it actually renders.

## Three tiers of "shipping a lens"

The same outcome is reachable at three levels of investment, each respecting a
different boundary. A given node can be satisfied by whichever tier fits.

### Tier 0 — canonical view-models (stack-free)

Preferred where it fits, and the only tier that touches **core**. A provider
parses its source at resolve time into a **pure-data view-model** — the first is
`CalendarModel` (events with start / end / all-day / summary / location /
category) — stores it on `node.data`, and declares a host-owned lens *by name*
(e.g. `viewer: 'calendar-month'`). **Zero render code is shipped.** These
view-models are data types and belong in core exactly like `KBGraph` does; the
host owns the components that render them. Tracked in
[core#76](https://github.com/anokye-labs/kbexplorer-core/issues/76) and consumed
by the ICS lenses ([template#495](https://github.com/anokye-labs/kbexplorer-template/issues/495)).

### Tier 1 — provider-shipped render contributions (`./views`)

When a type needs bespoke rendering the host can't provide generically, a
provider ships its own viewers / block renderers. Because the render contract is
**UX-stack-specific** (React + Fluent), it does *not* live in core. Instead it
lives in a published package,
**[`@anokye-labs/kbexplorer-view-kit`](https://github.com/anokye-labs/kbexplorer-template/issues/503)**
(a workspace subpackage of the template repo, with React as a peer dependency).
A provider's `./views` entry types against that package's versioned
`VIEW_API_VERSION` contract — *not* against unpublished template internals. The
data-only `.` entry stays clean; only render-capable hosts import `./views`.

### Tier 2 — site-local lenses (host repo `views/`)

A **site owner** — someone running their own content repo off the template — can
add a bespoke lens with a single **MDX/TSX view template** in their repo's
`views/` directory (never inside the replaceable `.kbx/` checkout), compiled by
the template build with the **highest** override precedence. One file in your own
repo, no package to publish. Tracked in
[template#504](https://github.com/anokye-labs/kbexplorer-template/issues/504).

## The contract change (additive, provider API 1.0.0 → 1.1.0)

All of it is additive-minor — existing data-only providers typecheck and load
unmodified, and incapable hosts degrade to data-only with **one warning, never a
rejection**. The canonical shapes live in
[`kbexplorer-core/src/provider.ts`](https://github.com/anokye-labs/kbexplorer-core/blob/main/src/provider.ts)
and
[`graph.ts`](https://github.com/anokye-labs/kbexplorer-core/blob/main/src/graph.ts);
this is the *design intent*, not the authority:

- **`ProviderModule.views?: string`** — a package-relative specifier (e.g.
  `'./views'`) for the module carrying the render contributions. A *specifier,
  not values*: importing `.` must never evaluate the render module graph. The
  views module's shape (`ProviderViews { viewers?, blockRenderers? }`) is defined
  by the view-kit, not core.
- **`ProviderCapability`** documents two new open-union members: `'viewers'` and
  `'block-renderers'`.
- **`KBNode.lenses?: NodeLens[]` and `defaultLens?: string`**, with
  `NodeLens { id; label?; viewer }` — a node advertises the named views it
  supports and which one is default.
- **Canonical view-models** (Tier 0), starting with `CalendarModel`.
- **`PROVIDER_API_VERSION`** bumps `'1.0.0' → '1.1.0'`.

### Degradation semantics

`capabilities` keeps its existing meaning — what a module **requires** —
and [`checkProviderCompatibility()`](https://github.com/anokye-labs/kbexplorer-core/blob/main/src/provider.ts)
is unchanged. The *presence* of a `views` declaration is the **optional offer**;
a provider that can run data-only must **not** list `'viewers'` in
`capabilities`. A host advertising `viewers` resolves and imports the views
entry; a host that doesn't loads the data half and skips the render half with one
warning. A lens-only provider that is useless without rendering *does* list
`'viewers'` and is cleanly skipped on incapable hosts — all of which already
falls out of the existing compatibility check.

## Choosing a lens

A node's default lens comes from `defaultLens`; the SPA and canvas expose a
**lens switcher** and honour a `?lens=` query parameter for deep-linking a
specific view ([template#493](https://github.com/anokye-labs/kbexplorer-template/issues/493)).
A **Source** lens is always available as the raw-data escape hatch. Selection is
by configuration / annotation, not code — the same node, many named views.
Viewer props gain **graph access** so a viewer can reach neighbours (e.g. a
GitHub PR viewer resolving its linked issue).

## Live blocks, same components

Block extraction already ships
([kbexplorer-cli#133](https://github.com/anokye-labs/kbexplorer-cli/pull/133));
its render half shipped for whole nodes in
[template#427](https://github.com/anokye-labs/kbexplorer-template/pull/427). This
epic closes the loop: the registry that loads provider `./views`
([template#492](https://github.com/anokye-labs/kbexplorer-template/issues/492))
feeds **both** a viewer registry (whole-file nodes) and a **block-renderer**
registry (fenced blocks), so a ` ```dot ` block and a `.dot` file node render
through the same live component. The reference provider implementation is
[kbexplorer-provider-rich-markdown#14](https://github.com/anokye-labs/kbexplorer-provider-rich-markdown/issues/14).

## Delivery plan

The epic is sequenced into three tracks. Track A is the platform (strictly
ordered on the core contract); Track B is the parallel lens/showcase work; Track
C is quality and delivery. See [#130](https://github.com/anokye-labs/kbexplorer/issues/130)
for live status.

### Track A — platform (sequenced)

| Item | Issue | What |
|---|---|---|
| A1 | [core#76](https://github.com/anokye-labs/kbexplorer-core/issues/76) | render-contribution contract (`views` entry) + `NodeLens` + canonical view-models (API 1.1.0) |
| A2 | [template#492](https://github.com/anokye-labs/kbexplorer-template/issues/492) | load provider `./views` into viewer + block registries (`render:true` gate) |
| A3 | [template#493](https://github.com/anokye-labs/kbexplorer-template/issues/493) | lens switcher + viewer props v2 (graph access, `?lens=` routing, Source lens) |
| A4 | [template#494](https://github.com/anokye-labs/kbexplorer-template/issues/494) | **bug** — canvas entry never registers builtin viewers |
| A5 | [template#503](https://github.com/anokye-labs/kbexplorer-template/issues/503) | publish `@anokye-labs/kbexplorer-view-kit` (render-contract package) |
| A6 | [template#504](https://github.com/anokye-labs/kbexplorer-template/issues/504) | site-local lenses — MDX/TSX view templates in the host repo (`views/`) |

### Track B — lenses & showcase (parallel)

| Item | Issue | What |
|---|---|---|
| B1 | [template#495](https://github.com/anokye-labs/kbexplorer-template/issues/495) | ICS lenses (month / agenda / Gantt) + live `ics` block |
| B2 | [template#496](https://github.com/anokye-labs/kbexplorer-template/issues/496) | Graphviz DOT live rendering (lazy WASM) + `dot` block |
| B3 | [template#497](https://github.com/anokye-labs/kbexplorer-template/issues/497) | animated flow lenses (DOT build-up + Mermaid sequence stepper) |
| B4 | [template#498](https://github.com/anokye-labs/kbexplorer-template/issues/498) | JSON Canvas `.canvas` ingestion + board lens + `canvas` block |
| B5 | [template#499](https://github.com/anokye-labs/kbexplorer-template/issues/499) | bespoke GitHub issue & pull-request viewers |
| B6 | [template#500](https://github.com/anokye-labs/kbexplorer-template/issues/500) | bespoke Wikipedia article viewer |
| B7 | [template#501](https://github.com/anokye-labs/kbexplorer-template/issues/501) | showcase sample nodes + structured-files tour page |
| B8 | [provider-rich-markdown#14](https://github.com/anokye-labs/kbexplorer-provider-rich-markdown/issues/14) | ship block renderers under the new contract (reference impl) |

### Track C — quality & delivery

| Item | Issue | What |
|---|---|---|
| C1 | [template#502](https://github.com/anokye-labs/kbexplorer-template/issues/502) | lens gallery + per-lens screenshot beauty gate + fallback-tier coverage report |
| C2 | [cli#252](https://github.com/anokye-labs/kbexplorer-cli/issues/252) | doctor diagnostics + canvas delivery verification for render-contributing providers |

## Scope boundaries & coordination

Referenced, not duplicated — these seams are owned elsewhere and this epic only
*uses* them:

- **Multi-lens is the unfiled clause of E6**
  ([template#426](https://github.com/anokye-labs/kbexplorer-template/issues/426));
  its rendering half shipped in
  [template#427](https://github.com/anokye-labs/kbexplorer-template/pull/427) and
  block extraction in [cli#133](https://github.com/anokye-labs/kbexplorer-cli/pull/133).
- **New structured-file *sources*** stay under the E2 generic-provider slot
  ([cli#132](https://github.com/anokye-labs/kbexplorer-cli/issues/132) /
  [#135](https://github.com/anokye-labs/kbexplorer-cli/issues/135)); this epic's
  ingestion pieces are demo/sample providers exercising that
  [Provider seam](architecture.md#bring-your-own-provider--via-defineprovider).
- **Engine render-free boundary** is respected: the engine binds render
  contributions via opaque host hooks; the engine-repo counterpart is tracked
  under the [#472](https://github.com/anokye-labs/kbexplorer-template/issues/472)
  extraction program.
- **Canvas program** ([kbexplorer#8](https://github.com/anokye-labs/kbexplorer/issues/8)):
  [template#407](https://github.com/anokye-labs/kbexplorer-template/issues/407)
  reuses node-type viewers, so provider-shipped lenses reach the copilot surface
  via the template build — the CLI loopback server serves the template's
  `dist/`, with no CLI bundling step ([the two surfaces](surfaces.md)).
- **Block→node promotion** stays out of scope — it remains the open question in
  [decisions/markdown-rendering-default.md](decisions/markdown-rendering-default.md)
  (option c), as do org-chart lenses, MDX / Adaptive Cards inputs, and the
  canvas `anchor` action `lens` argument.

## Success definition

The public demo shows one `.ics` node through calendar / agenda / Gantt lenses
via a switcher; a DOT node as live graph *and* animated flow; a `.canvas` board;
and richer GitHub issue / PR / Wikipedia views — each scoring ≥3 on every
[`BEAUTY.md`](https://github.com/anokye-labs/kbexplorer-template/blob/main/BEAUTY.md)
dimension in the captured review set. Every file node in the template's own graph
renders at **tier 1 (bespoke)** or **tier 2 (shape-inferred)**, proven by a
coverage report. A third-party provider earns a bespoke lens on both surfaces
with `npm install` + one `config.yaml` entry — and a site owner earns one with a
single MDX file in their own repo.

---

> **Layer context.** Render contributions extend
> [Layer 4 — Representation](architecture.md#layer-4--representation) and the
> [extension points](architecture.md#extension-points); the contract is the
> canonical one in
> [`kbexplorer-core`](https://github.com/anokye-labs/kbexplorer-core), and the
> UX-stack render contract lives in
> [`@anokye-labs/kbexplorer-view-kit`](https://github.com/anokye-labs/kbexplorer-template/issues/503).
> Program placement: [roadmap.md](roadmap.md); parent portfolio item
> [kbexplorer#13](https://github.com/anokye-labs/kbexplorer/issues/13).
