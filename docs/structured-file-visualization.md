# Structured-file visualization — provider-shipped lenses and representations

**Status:** Proposed design, ready to build. This document fleshes out the draft
"A View of Each File" requirements into a grounded, cross-repo design and work
breakdown. Requirements anchor:
[kbexplorer#12](https://github.com/anokye-labs/kbexplorer/issues/12) — near-term
outcomes **O2** (one Markdown doc mixing frontmatter + links + prose + embedded
rendered blocks) and especially **O3** ("More structured file types, and one
dataset rendered many ways": an `.ics` schedule → a **calendar** *or* a
**Gantt**; a DOT / JSON-Canvas graph → an **org chart** *or* a **board**; the
representation a dataset takes is a **configuration/annotation concern**).

This program delivers the unfiled "lenses / multi-lens-by-annotation" half of
E6 ([kbexplorer-template#426](https://github.com/anokye-labs/kbexplorer-template/issues/426)
— whose rendering half shipped in template#427) and fills the gap the issue
audit found: **no epic owns O3's "one dataset, many representations" table, no
contract exists for provider-shipped viewers, and no open issue tracks a
Graphviz/ICS/JSON-Canvas/Gantt/GitHub-issue/Wikipedia viewer as a named
deliverable.**

Tracking epic:
[anokye-labs/kbexplorer#130](https://github.com/anokye-labs/kbexplorer/issues/130)
(Part of the foundation program #13), with fourteen sub-issues across four
repos — see [Work breakdown](#work-breakdown).

---

## 1. Capability statement

Every file surfaced by KB Explorer should resolve to a real, typed
visualization when opened on any KBX surface — not a raw dump. The **same**
structured resource should be viewable through **multiple named lenses** (an
`.ics` file as a month calendar, an agenda, or a Gantt chart), chosen by
configuration/annotation rather than code forks. Structured content embedded as
fenced blocks inside Markdown should render **live** through the same lens
components as whole-file nodes. And external provider packages must be able to
deliver **both halves** of a new type: the ingestion that maps structured files
into nodes, *and* the lenses that render those nodes — including contributing
new lenses for resources that **other** providers brought in (GitHub issues,
pull requests, Wikipedia pages).

## 2. Terminology

| Term | Meaning | Where it lives |
| --- | --- | --- |
| **Representation** | Whole-graph render target (`spa`, `copilot`, `json-ld`, `llm-context`) — the existing `Representation<Out>` seam | `kbexplorer-core` `src/representation.ts` (unchanged by this program) |
| **Viewer** | A React component keyed into the viewer registry by `entityType` / JSON-LD `@type` / explicit key | template `src/views/viewers/registry.ts` |
| **Lens** | A *named per-node view*: `{ id, label, viewer }`. One node may carry several; exactly one is default | **new**, `KBNode.lenses` (core, additive) |
| **Block renderer** | Maps a fenced-block `kind` to a render decision (`mermaid` / `svg` / `unsupported`, extended below) | template `src/views/rich-markdown/registry.ts` |
| **Render contribution** | The render half a provider may ship: a `./views` entry point carrying viewers + block renderers, declared via `ProviderModule.views` | **new**, `ProviderModule` (core, additive) |
| **View-kit** | The published render contract the views entry types against (`ViewerProps`, `BlockOutput`, `VIEW_API_VERSION`); React peer dep | **new**, `@anokye-labs/kbexplorer-view-kit`, a workspace subpackage of the template repo |
| **Canonical view-model** | Pure data contract for a host-owned model lens (e.g. `CalendarModel`) — the stack-free render contribution | **new**, core (additive) |
| **Site lens** | A site-owner-authored MDX/TSX view template in the host repo (`views/`), compiled by the template build, highest override precedence | **new**, host repo (machinery in template) |

## 3. Current state (verified in-repo, 2026-07-04)

The data side is open and extensible; the render side is host-only. Verified
against the working trees of all six sibling repos plus the engine package
pinned by the template (`kbexplorer-engine@23760f3`, slices 1–2 of the #472
extraction landed):

| Seam | Where today | Externally extensible? |
| --- | --- | --- |
| File → node (declarative rules + shape inference) | `structured-node-map.yaml` + `StructuralProvider`, `applyStructuredNodeMap` / `inferStructuredNode` — **engine package** since template#491 | Yes (YAML) — but parses only YAML/JSON content |
| File → node (schema spine) | `content-model/` + `ContentModelProvider` — engine package | Yes (drop entity YAML) |
| Custom ingestion | `defineProvider()` factories via `config.yaml` `providers[].module`, loaded by engine `loadExternalProviders` | Yes — the existing contract (core#9) |
| Node → viewer routing | template `src/views/viewers/registry.ts` (`resolveViewer`: node-type `viewer` key → `entityType` → JSON-LD `@type` → `GenericStructuredView`; case-insensitive, last-registration-wins) | **No** — components bound only in `src/views/viewers/builtin-map.ts`, called from `src/main.tsx` |
| Fenced-block rendering | template `src/views/rich-markdown/registry.ts` (`BlockOutput = mermaid \| svg \| unsupported`); live rendering exists **only for Mermaid**; `dot`/`ics`/`canvas` render only from provider-shipped pre-built SVG | **No** — host-only |
| Per-node multi-view | — | **Does not exist.** One `display` mode per node, no switcher, no lens model |
| Provider compat | `PROVIDER_API_VERSION = '1.0.0'`, open `ProviderCapability` union, `checkProviderCompatibility`; engine host contract advertises `['graph:nodes','graph:edges']` | Mechanism exists; no render-side capability names |

Bespoke viewers that exist today: 14 components / 15 keys, all for the
content-model spine and `.github` structural kinds (person, squad, workflow,
action, skill, …). **There is no GitHub-issue, pull-request, commit,
repository, release, or Wikipedia viewer** — those nodes render as prose with a
`SourceBadge`. The Wikipedia provider (18 articles in the template's
`content/config.yaml`) and WorkProvider (193 issues / 170 PRs in the shipped
demo) produce rich `data` that no viewer consumes.

Known defect this design fixes in passing: the embeddable canvas entry
(`src/canvas.tsx`) never calls `registerBuiltinViewers()` — only `main.tsx`
does — so on the `copilot` surface every node falls back to
`GenericStructuredView` despite `AnchorFirstView`'s stated intent to reuse the
bespoke viewers.

Corrections to the draft requirements doc, from this audit:

- `docs/compatibility.md` **does not exist** in `kbexplorer-cli`; the
  CLI↔template compatibility story is `.kbx.json` ref-pinning + the (not yet
  template-advertised) capability sidecar checked by `kbx doctor`
  (template#416/#265/#417/#418).
- The engine pin is `23760f3` (slice 2/5 merged), not `435226f` (slice 1);
  providers, plugin-loader, content-model and structured-node-map all now live
  in the **engine package**, so the loader-side work below is engine work.
- The rich-markdown provider's `canvas` block kind is currently a *generic*
  drawing-canvas placeholder; nothing in any repo implements the
  [JSON Canvas](https://jsoncanvas.org) file format.

## 4. The model: three provider contribution modes

A provider package may contribute any combination of:

1. **Lens-only, for existing resources.** The module ships viewers keyed to
   types that *other* providers emit (e.g. richer `issue` / `pull-request`
   viewers over WorkProvider nodes, or a `wikipedia-article` card over
   WikipediaProvider nodes). No nodes of its own. This is the genuinely new
   architectural ground: today `adaptCoreProvider` in the engine's plugin
   loader discards every module export except the factory result.
2. **Resource + lens.** The module ingests a new structured file type into
   nodes *and* ships the lenses that render them (e.g. an ICS provider emitting
   `calendar` nodes plus month/agenda/Gantt viewers). This is the "both halves"
   requirement (FR-1 of the draft).
3. **Lens with a resource dependency.** A lens whose rendering needs *other*
   nodes: a Gantt over a milestone plan needs the issue nodes it schedules; an
   org-chart lens needs the person/team spine. Build-time ordering already
   exists (`GraphProvider.dependencies` → `ProviderRegistry.getExecutionOrder()`
   topo-sort; `ProviderContext.existingNodes`); what is missing is **render-time
   graph access for viewers**, added in §7.

## 5. Contract changes (kbexplorer-core, additive, minor)

`PROVIDER_API_VERSION`: `1.0.0` → `1.1.0`.

```ts
// provider.ts — documented capability names added to the open union
export type ProviderCapability =
  | 'graph:nodes' | 'graph:edges' | 'sources'
  | 'viewers'            // host binds ProviderModule.viewers into its viewer registry
  | 'block-renderers'    // host binds ProviderModule.blockRenderers into its block registry
  | (string & {});

export interface ProviderModule {
  default: ProviderFactory;
  apiVersion?: string;
  capabilities?: ProviderCapability[];   // unchanged: capabilities the module REQUIRES
  /** Render contribution: package-relative specifier of the module carrying
   *  viewers + block renderers (e.g. './views'). A SPECIFIER, not values —
   *  importing the '.' entry must never evaluate the render module graph. */
  views?: string;
}
```

**Why a specifier and not exported values.** The CLI's composite ingest runs in
Node with no DOM and no React; the engine is render-free by boundary test. If
viewer components were values on the data module, every data-only host would
evaluate React the moment it imports the provider — the capability check can
gate *binding*, but not *evaluation* of an already-imported module graph. With
a declared entry point, only hosts that advertise the capability (and whose
config opts in) ever dynamic-import the render half; the `.` entry stays
core-only and loadable everywhere.

**Degradation semantics (FR-2), with no change to `checkProviderCompatibility`:**
`capabilities` keeps its existing meaning — capabilities the module *requires*.
A provider that can run data-only must **not** list `'viewers'` there; the
presence of the `views` declaration *is* the optional offer. A host that
advertises `'viewers'` in its `ProviderHostContract` imports and binds the
views entry; a host that doesn't logs one warning and loads the data half —
nodes still render via `GenericStructuredView`. A provider that is *useless*
without its render half (mode-1 lens-only packages) lists `'viewers'` in
`capabilities` and is skipped outright on incapable hosts — both behaviors fall
out of the existing check.

**Canonical view-models — the stack-free tier (preferred where it fits).**
Most of what a "calendar provider" contributes to rendering isn't a component,
it's a *parse to a known shape*. Core gains small pure data contracts for
host-owned model lenses — first `CalendarModel` (events with start/end/all-day/
summary/location/category); later `GraphModel`, `BoardModel`, `TableModel` as
the lens catalog proves them. A provider parses at `resolve()` time, stores the
model on `node.data`, and declares `viewer: 'calendar-month'` — **zero render
code shipped**, works on every surface, testable under `node:test`. The
pre-built-SVG fallback remains the universal escape hatch below that
(mime-bundle-style degradation). Bespoke components are the escalation path,
not the default.

```ts
// graph.ts — per-node lens model (additive optional fields on KBNode)
export interface NodeLens {
  /** Stable lens id, unique within the node (e.g. 'calendar', 'agenda', 'gantt'). */
  id: string;
  /** Human label for the switcher. Defaults to the id. */
  label?: string;
  /** Viewer-registry key that renders this lens. */
  viewer: string;
}
// KBNode gains:
//   lenses?: NodeLens[];
//   defaultLens?: string;   // id of the lens to open with; falls back to lenses[0]
```

Nodes without `lenses` behave exactly as today (viewer resolution via
node-type `viewer` key → `entityType` → `@type` → generic). This satisfies
core's additive policy: every exported change is a new optional field or a new
documented member of an open union.

**Where lens annotations come from** (the O3 "configuration/annotation
concern"): (a) `structured-node-map.yaml` rules gain a `lenses:` field; (b)
frontmatter on authored/rich-markdown docs; (c) provider code. All three
converge on the same `KBNode.lenses` data.

**The render contract's home: `@anokye-labs/kbexplorer-view-kit`.** A viewer is
inherently UX-stack-specific (React + Fluent), so its contract cannot live in
core (dependency-free) or the engine (render-free). It lives where the stack
lives: a **published workspace subpackage of the template repo** exporting
`ViewerProps`, `ViewerComponent`, `LazyViewer`, `BlockRenderer`, `BlockOutput`,
the `ProviderViews` module shape, and a `VIEW_API_VERSION` (versioned
independently, same-major-compatible) — with **React as a peer dependency**.
The template consumes its own package so the types are single-sourced and
version in lockstep with the surfaces that satisfy them. The dependency
picture then has no layer-skips: data half → core; canonical parsers → core
view-model types; bespoke lenses → view-kit; template → composes all of it.
No-build-step providers (NFR-4) write `React.createElement` (view-kit as a
types-only devDependency) or precompile just their views entry — the `.` data
entry stays pure ESM either way.

## 6. Loading & registration flow (engine + template)

The engine stays render-free. `loadExternalProviders` (engine
`src/plugin-loader.ts`) gains an optional host-hooks parameter:

```ts
loadExternalProviders(configs, hooks?: {
  registerViewer?(key: string, viewer: unknown): void;
  registerBlockRenderer?(kind: string, renderer: unknown): void;
})
```

After the existing compat check, when the host contract advertises the
matching capability *and* the entry opts in, the loader resolves `mod.views`
relative to the provider's specifier (same `classifySpecifier` policy),
dynamic-imports it, and hands the loaded `ProviderViews` object to the hooks
as **opaque values** — the engine never imports React, and on incapable or
non-opted-in hosts the views module is never evaluated at all.
Registration order = `config.yaml` declared order (already the loader's
iteration order), and both registries are last-registration-wins, so downstream
packages can override built-ins deterministically (FR-3). This mirrors the
established #467 inversion ("engine declares viewer *names*, host binds
components") rather than fighting it.

Template side: the single call site (`src/engine/loader.ts` →
`registerProviders`) passes `registerViewer` from `src/views/viewers/registry`
and `registerBlockRenderer` from `src/views/rich-markdown/registry`. Built-in
viewer registration moves out of `main.tsx` into a shared composition module
invoked by **both** entries (`main.tsx` and `canvas.tsx`) — fixing the canvas
asymmetry and guaranteeing surface parity (FR-6). Because `canvas.html` is a
second Vite entry of the *same template build* and the CLI's loopback server
just serves that `dist/` (verified: `defaultResolveBuildDir` in
`kbexplorer-cli/src/extension/canvas/state.js` — no CLI-side bundling exists),
provider viewers reach the copilot surface with **zero CLI changes** as long as
the provider is a dependency of the template checkout that got built. `kbx
doctor` gains a check that render-contributing providers in the host site are
actually resolvable in `.kbx/`.

**Lazy loading (open question 4, resolved):** contribution values may be either
a component or a zero-arg loader returning `Promise<{ default: Component }>`.
The template registry accepts both; lazy entries resolve on first render behind
`React.lazy`/`Suspense`. Heavy engines (WASM Graphviz) must use the lazy form
so the SPA's initial bundle is unaffected.

### Site-local lenses — MDX/TSX view templates in the host repo

The third contribution tier, for site owners rather than package authors:
bespoke per-repo (or per-folder) renderings authored as files in the **host
repo** — `views/**/*.{mdx,tsx}` next to `content/` — with no package to
publish. Placement is the load-bearing decision: authored templates must live
in the host repo, **never inside the `.kbx/` template checkout**, because that
checkout is a pinned submodule/vendor copy that `kbx update` replaces — edits
there are lost. The *machinery* lives in the template repo: a Vite plugin
discovers host `views/` files through the same `VITE_KB_HOST_ROOT` seam the
manifest generator uses, compiles MDX via `@mdx-js/rollup` (TSX comes free via
esbuild), and registers the resulting components — a no-op for sites without a
`views/` directory. In self-hosted mode (the template repo *is* the site, as
in the demo) host root = template root and `views/` at the repo root works
identically. Docs call these **site lenses** to avoid overloading "template".

An MDX site lens is markdown with component slots, receiving the node and its
view-model as props, with the host's canonical model lenses in scope:

````mdx
---
lens: team-calendar
appliesTo: { entityType: calendar, path: "team/**" }
---
# {props.node.title}

<CalendarMonth model={props.model} accent="brand" />

Our ceremonies — updated weekly, times in PT.
````

Registration order makes precedence most-specific-wins via the existing
last-registration-wins registries: **builtins → provider `./views` entries
(config order) → site lenses**. Scoping stays data: `appliesTo` resolves
through the same lens-assignment pass as `structured-node-map.yaml` rules,
stamping `lenses`/`defaultLens` onto matching nodes at build time — the
registry never learns about globs. Both surfaces get site lenses for free
(same Vite build), and the gallery/screenshot harness (§13) picks them up so
hand-authored lenses meet the same beauty bar. Trust-wise this is the mildest
tier — the host repo's own code on the host's own site, the same grant as the
existing repo-local provider modules and theme modules. This also serves as
the prototype for `docs/decisions/markdown-rendering-default.md` option (b):
rendering as a property of the site/folder rather than the document.

**Trust (NFR-1):** the specifier policy is unchanged (`classifySpecifier`:
relative-local and bare npm names only; absolute paths and URL schemes
rejected). Executing a provider's render half is executing third-party React in
the page — the same grant as installing any dependency, but to make it
deliberate, a provider entry must opt in with `render: true` in its
`config.yaml` entry; without it the host skips the render half with the same
single warning as an incapable host.

## 7. Viewer props v2 — render-time graph access

Today `ViewerProps` is exactly `{ node: KBNode }`; a viewer cannot read other
nodes (dependencies are impossible to render). Additive extension, template
side:

```ts
export interface ViewerProps {
  node: KBNode;
  /** The pure KBGraph (read-only), for lenses that render dependencies. */
  graph?: KBGraph;
  /** Active lens id when the node carries lenses. */
  lens?: string;
}
```

`ReadingView` and `AnchorFirstView` (the only two call sites) pass `graph`.
Existing viewers ignore the new props. A Gantt lens finds its issue nodes via
`graph.nodes` + the plan node's connections; an org-chart lens walks
`reports-to` edges. Viewers remain pure render functions over the pure graph —
the purity boundary (representations never reach into the engine) is untouched.

## 8. Lens switcher UX

When `node.lenses?.length > 1`, the reading surface shows a compact switcher
(pill row / dropdown in the node header) selecting among lens labels; the
active lens is reflected in the URL (`#/node/<id>?lens=<lensId>`) so links can
target a specific view, and the choice defaults to `node.defaultLens`. On the
copilot surface, `AnchorFirstView` renders the default lens and exposes the
alternates through the existing view-action seam (an `anchor` action with a
`lens` argument is a natural later extension of the canvas `actions[]`). Every
node with `sourceFile`/`rawContent` additionally gets an always-available
**Source** lens (raw view) — closing the "no raw toggle" gap and making the
fallback tier explicit and user-visible.

## 9. Embedded blocks — live rendering through the same lenses

`BlockOutput` (template block registry) gains one member:

```ts
| { type: 'viewer'; key: string; data: unknown; title?: string }
```

A block renderer may now parse the fence source and delegate to a
viewer-registry key — so an ` ```ics ` block renders with the **same**
agenda/calendar components as a whole `.ics` file node, and an ` ```canvas `
block with the same board component as a `.canvas` file node ("one renderer,
two mounts"). Precedence keeps the existing guarantees: registered renderer
decision first; provider-shipped pre-built SVG still wins over `unsupported`;
raw source remains the last resort. The existing `BlockRenderContext.isDark`
flag finally gets consumers (theme-aware live rendering).

Block **kinds** stay aligned with the rich-markdown provider's frozen allowlist
(`dot`, `mermaid`, `ics`, `canvas` — `RICH_MARKDOWN_BLOCK_LANGS`); widening
that list (e.g. `csv`, `jsoncanvas` alias) is an additive provider-package
change that rides the same contract.

## 10. Fallback tiers and coverage (FR-5)

Explicit, observable tiers, reported per node:

1. **Tier 1 — bespoke lens** (registered viewer for a declared lens or type)
2. **Tier 2 — shape-inferred structured view** (`inferStructuredNode` typing +
   `GenericStructuredView`)
3. **Tier 3 — raw file view**

The engine's structured-file path already guarantees tier ≥ 2 for YAML/JSON;
the gap is text formats (`.ics`, `.dot`, `.canvas` is JSON so it parses today)
and `nodemap.yaml` plain-file nodes that skip shape inference. The showcase
assessment script (`assess-graph`) gains a per-tier count so "view of each
file" regressions are measurable, and acceptance for the program is: every
file node in the template's own repo graph renders at tier 1 or 2, raw-file
fallback only for genuinely unstructured content.

## 11. The lens catalog (what we're actually going to look at)

Each of these is a feature with its own visual acceptance bar (§12). Data
shapes verified against what providers already emit.

- **ICS → calendar lenses.** Parse a pragmatic RFC 5545 subset (VEVENT:
  DTSTART/DTEND/SUMMARY/LOCATION/UID; recurrence expansion out of scope v1,
  surfaced as a labeled limitation). Lenses: **month grid**, **agenda list**,
  **Gantt** (events as horizontal spans grouped by calendar/category). Live
  ` ```ics ` block renderer replaces today's SVG-only fallback.
- **DOT → live graph + animated process flow.** Live Graphviz rendering via a
  lazily-loaded WASM engine (e.g. `@hpcc-js/wasm-graphviz`) replacing the
  SVG-only fallback; then an **animated flow lens**: stage the rendered digraph
  topologically and draw nodes/edges in sequence (CSS keyframes, respects
  `prefers-reduced-motion`) — the same build-up idiom as this repo's
  hand-authored pipeline SVGs (docs/graph-build), now derived from DOT source
  instead of hand-drawn.
- **Animated sequence step-through.** Mermaid sequence diagrams already render
  live; add a stepper lens (play/pause/step controls highlighting one message
  at a time), sharing the animation-stepper primitive with the DOT flow lens.
- **JSON Canvas → board lens.** Implement the actual
  [JSON Canvas](https://jsoncanvas.org) spec (`.canvas`: text/file/link/group
  nodes + edges) as a pan/zoom board; `file` nodes that match graph nodes
  become navigable `#/node/<id>` links. Ingestion: a `structured-node-map.yaml`
  shape rule (`nodes` + `edges`) — `.canvas` content is JSON and already
  parses.
- **GitHub issue & pull-request viewers.** Bespoke entity views over the
  WorkProvider data that already ships (number, title, body, state, labels
  with colors, assignees, timestamps, `head_branch`): state chip, label pills
  (using label colors with contrast-safe text), sanitized body prose,
  cross-referenced issues/PRs as navigable edges, PR branch/merge facts.
  Today these render as plain prose with a badge.
- **Wikipedia article viewer.** Summary card over the WikipediaProvider data
  (title, description, extract, thumbnail, canonical link): thumbnail-led
  card, sanitized extract, "read on Wikipedia" affordance, connected concepts.
- **Org-chart lenses (later wave).** The O3 "reporting vs operating" pair over
  the existing content-model spine, reusing the `reports-to` layout that
  already exists in the graph canvas.

## 12. Showcase plan (template demo site)

New sample content in `kbexplorer-template` exercising every lens on the
public Pages demo, all reachable from a "Structured files tour" rich-markdown
page that also embeds each kind as a fenced block:

- `content/samples/team-calendar.ics` — a plausible team calendar (cycle
  ceremonies, releases) → calendar/agenda/Gantt lenses.
- `content/samples/release-flow.dot` — the repo's release process as a digraph
  → graph + animated flow lenses.
- `content/samples/planning-board.canvas` — a JSON Canvas planning board whose
  file nodes point at real graph nodes.
- Existing issues/PRs/Wikipedia nodes light up via the new viewers with zero
  content changes.

Ingestion for the text formats (`.ics`, `.dot`) lands as sample providers
following the `examples/quotes-provider` pattern, making them the runnable
render-contribution examples `docs/providers.md` needs (FR-7) — with
`kbexplorer-provider-rich-markdown` shipping its block renderers as the
published reference implementation.

## 13. Standard of beauty — visual verification

Grounded in the template's existing apparatus (BEAUTY.md 7-dimension rubric,
`capture:review` 9-surface × 4-theme × 2-viewport screenshot harness,
pixelmatch nightly gate, property-based `audit:visual`):

1. **Lens gallery route** — a dev-gated route (`#/gallery/lenses`) rendering
   every registered viewer and block renderer against committed fixture nodes,
   one card per (viewer × fixture). This is the missing isolated-viewer mount
   the capture harness needs.
2. **Per-lens capture surfaces** — gallery entries added to
   `scripts/review-surfaces.json`, inheriting the theme × viewport matrix; new
   baselines committed deliberately, diffed nightly like everything else.
3. **Beauty review per lens** — each viewer feature's acceptance includes a
   BEAUTY.md scoring pass (≥3 on all seven dimensions) over the captured
   images, recorded in the feature issue.
4. **Property audits on the PR path** — cheap invariants for every gallery
   card: no horizontal overflow at 390px, non-blank render, zero console
   errors, `prefers-reduced-motion` honored for animated lenses.
5. **Coverage-tier report** (§10) wired into `assess`.

## 14. Versioning, compatibility, testing

- Core: one minor release (`PROVIDER_API_VERSION` 1.1.0 + `NodeLens` +
  `ProviderModule.views` + first canonical view-models); data-only providers
  load unmodified; contract tests extend `test/provider-compat.test.ts` with
  the two capability names and the degrade-don't-reject behavior.
- View-kit: released from the template repo with its own `VIEW_API_VERSION`
  (same-major-compatible); a provider's views entry declares the version it
  targets, checked at bind time with the same skip-and-warn semantics.
- Engine/template: loader tests for registration order, capability-gated skip,
  `render: true` gating, and both-entries viewer registration; the boundary
  test keeps React out of the engine (hooks receive opaque values).
- CLI: `kbx doctor` provider-render diagnostics; the compatibility policy row
  lands in the template-advertised capability sidecar work already tracked
  (template#416/#265) rather than a new matrix document.
- Provider repos: the module-contract `node:test` suites grow a
  render-contribution section (exports shape, lazy-loader form, frozen maps).

## 15. Out of scope

- New whole-graph `Representation` targets; the relation taxonomy; the
  two-identifier identity model; the content-model spine.
- Marketplace/auto-discovery: loading stays explicit via `config.yaml`.
- Recurrence expansion (RRULE) in v1 ICS; MDX and Adaptive Cards from O3's
  longer list (later waves once the contract is proven on four formats).
- Block→node promotion (`docs/decisions/markdown-rendering-default.md` option
  c) — this design keeps blocks anonymous but makes the lens components
  reusable, which is the prerequisite either way.

## Work breakdown

Three tracks; Track B features are mutually parallel by design.

**Track A — platform (sequenced):**
| # | Issue | Feature |
| --- | --- | --- |
| A1 | [core#76](https://github.com/anokye-labs/kbexplorer-core/issues/76) | Render-contribution contract (`views` entry declaration) + `NodeLens` + canonical view-models (API 1.1.0) |
| A2 | [template#492](https://github.com/anokye-labs/kbexplorer-template/issues/492) | Load provider `./views` entries into both registries; `render: true` gate; shared entry composition |
| A3 | [template#493](https://github.com/anokye-labs/kbexplorer-template/issues/493) | Lens switcher + viewer props v2 (graph access, `?lens=` routing, Source lens) |
| A4 | [template#494](https://github.com/anokye-labs/kbexplorer-template/issues/494) | Bug: canvas entry never registers builtin viewers |
| A5 | [template#503](https://github.com/anokye-labs/kbexplorer-template/issues/503) | Publish `@anokye-labs/kbexplorer-view-kit` — the render-contract package (React peer dep) |
| A6 | [template#504](https://github.com/anokye-labs/kbexplorer-template/issues/504) | Site-local lenses — MDX/TSX view templates in the host repo (`views/`) |

**Track B — lenses & showcase (parallel):**
| # | Issue | Feature |
| --- | --- | --- |
| B1 | [template#495](https://github.com/anokye-labs/kbexplorer-template/issues/495) | ICS lenses: month / agenda / Gantt + live `ics` block |
| B2 | [template#496](https://github.com/anokye-labs/kbexplorer-template/issues/496) | Graphviz DOT live rendering + `dot` block |
| B3 | [template#497](https://github.com/anokye-labs/kbexplorer-template/issues/497) | Animated flow lenses: DOT process flow + Mermaid sequence stepper |
| B4 | [template#498](https://github.com/anokye-labs/kbexplorer-template/issues/498) | JSON Canvas: `.canvas` ingestion + board lens + `canvas` block |
| B5 | [template#499](https://github.com/anokye-labs/kbexplorer-template/issues/499) | Bespoke GitHub issue & pull-request viewers |
| B6 | [template#500](https://github.com/anokye-labs/kbexplorer-template/issues/500) | Bespoke Wikipedia article viewer |
| B7 | [template#501](https://github.com/anokye-labs/kbexplorer-template/issues/501) | Showcase: structured-file sample nodes + tour page |
| B8 | [provider-rich-markdown#14](https://github.com/anokye-labs/kbexplorer-provider-rich-markdown/issues/14) | Ship block renderers under the new contract (reference implementation) |

**Track C — quality & delivery:**
| # | Issue | Feature |
| --- | --- | --- |
| C1 | [template#502](https://github.com/anokye-labs/kbexplorer-template/issues/502) | Lens gallery + per-lens screenshot beauty gate + coverage tiers |
| C2 | [cli#252](https://github.com/anokye-labs/kbexplorer-cli/issues/252) | Doctor diagnostics + canvas delivery verification for render-contributing providers |

Engine-repo counterpart work (the `loadExternalProviders` hooks) is called out
in A2's body; the engine repo is tracked separately under the #472 extraction
program.

## Acceptance criteria (program level)

1. `npm install` of a provider package + one `config.yaml` entry (with
   `render: true`) yields a bespoke lens for that provider's nodes on both
   `spa` and `copilot` surfaces, with zero host-repo code changes.
2. The same provider on a host without the `'viewers'` capability still
   contributes nodes/edges, rendering via `GenericStructuredView`, with a
   single logged warning.
3. The demo site shows the same `.ics` node through calendar, agenda, and
   Gantt lenses via the switcher; a DOT node as a live graph and an animated
   flow; a `.canvas` board; and richer GitHub issue / PR / Wikipedia views —
   each scoring ≥3 on every BEAUTY.md dimension in the captured review set.
4. Every file node in the template's own repo graph renders at tier 1 or 2,
   proven by the coverage report.
5. `docs/providers.md` documents the render-side authoring path with two
   runnable examples; core's changelog records the additive 1.1.0 bump.
