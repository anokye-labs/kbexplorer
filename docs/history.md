# History — a verified, chronological account

This is a dated changelog across all five repos in the kbx system
(`kbexplorer-core`, `kbexplorer-cli`, `kbexplorer-template`,
`kbexplorer-search`, and this docs/showcase repo, `kbexplorer`), built by
reading actual PR/issue bodies and merge timestamps — not by summarizing prior
notes. Where [roadmap.md](roadmap.md) shows the *structural* plan (epics and
waves), this page shows the *order things actually shipped in*.

Every entry below is sourced from a `gh pr view` / `gh issue view` /
`git log` / `git tag` lookup against the live repos. Timestamps are merge
times (`mergedAt`), not commit author dates, which can differ.

> **Scope note.** This is not exhaustive — dozens of smaller PRs shipped
> between the milestones below. It tracks the capability-defining moments the
> rest of this doc set (architecture, surfaces, decisions) refers back to.

## Foundation — `kbexplorer-core` contracts

`kbexplorer-core` ships no application code — only the dependency-free
contracts every other repo imports. It cut three tagged releases in quick
succession:

| Release | Tagged | Notable contracts added |
|---|---|---|
| [`v0.1.0`](https://github.com/anokye-labs/kbexplorer-core/releases/tag/v0.1.0) | 2026-06-30 07:52 -0400 | Configurable identity scheme/authority, opaque type-independent address body, alias-based person identity, presentation-token contract, canvas-as-Representation-target convention, SourceRef/provenance, observed-vs-derived. |
| [`v0.2.0`](https://github.com/anokye-labs/kbexplorer-core/releases/tag/v0.2.0) | 2026-06-30 15:42 -0400 | Identity claims + `linkedRefs`, source-precedence config, `relationRaw` passthrough. |
| [`v0.3.0`](https://github.com/anokye-labs/kbexplorer-core/releases/tag/v0.3.0) | 2026-06-30 17:00 -0400 | Access-label field on nodes/edges (E5). |

A fourth contract — redaction + access-review — merged to `main` after `v0.3.0`
but is **not yet tagged as a release** as of this writing.

## The rich-Markdown slice — Mermaid, block rendering, and authored ingestion (three distinct capabilities)

These three shipped on consecutive days, in this order, and are easy to
conflate because they all touch "Markdown rendering." They are not the same
change:

1. **2026-06-29 19:02:57Z** — [`kbexplorer-template#422`](https://github.com/anokye-labs/kbexplorer-template/pull/422)
   *"Render Mermaid diagrams in ReadingView"* (closes
   [#417](https://github.com/anokye-labs/kbexplorer-template/issues/417)).
   Representation-layer only: `ReadingView`'s `display: diagram` nodes and
   fenced `mermaid` code blocks render live, with a visible raw-source
   fallback for invalid/unsupported diagrams. This is the **first** live
   diagram rendering in the reading view.
2. **2026-06-30 12:56:57Z** — [`kbexplorer-template#430`](https://github.com/anokye-labs/kbexplorer-template/pull/430)
   *"Rich-Markdown rendering with block-renderer registry"* (closes
   [#427](https://github.com/anokye-labs/kbexplorer-template/issues/427)).
   Adds an open block-renderer registry, a `RichMarkdownDocumentView`
   (frontmatter facts + prose + blocks), reuses #422's inline Mermaid path as
   the one *live* renderer, and adds a **pre-built-SVG fallback** for
   `dot`/`ics`/`canvas` blocks so nothing falls back to raw code when an SVG
   exists. At merge, this only fired on the `?demo=richmd` sample route — no
   authored content used it yet.
3. **2026-06-30 17:50:23Z** — [`kbexplorer-template#432`](https://github.com/anokye-labs/kbexplorer-template/pull/432)
   *"Authored rich-Markdown integration"* (closes
   [#431](https://github.com/anokye-labs/kbexplorer-template/issues/431)).
   Wires the published
   [`@anokye-labs/kbexplorer-provider-rich-markdown`](https://github.com/anokye-labs/kbexplorer-provider-rich-markdown)
   package's `ingestRichMarkdown` (pinned `v0.1.0`) into the engine via a new
   `AuthoredRichMarkdownProvider`. A doc opts in with `display: rich-markdown`
   in its YAML frontmatter — the discriminator lives in
   [`src/engine/providers/rich-markdown/detect.ts`](https://github.com/anokye-labs/kbexplorer-template/blob/main/src/engine/providers/rich-markdown/detect.ts).
   This is the change that makes #430's rendering reachable from **real,
   authored content** for the first time.

`kbexplorer-template v0.3.0` tagged at **2026-06-30 18:10:17Z** carries both
#430 and #432.

### The showcase lights it up the same day

[`kbexplorer#95`](https://github.com/anokye-labs/kbexplorer/pull/95) *"showcase:
render this repo as the KB host with a rich-Markdown demo"* merged
**2026-06-30 18:16:19Z** — six minutes after the template tag. It bumped this
repo's `TEMPLATE_REF` from `v0.2.0` to `v0.3.0`, wired `VITE_KB_HOST_ROOT` to
this repo, and added `content/org/platform.md` with `display: rich-markdown`
in its frontmatter. **This file is still in the repo and still deployed** —
the live showcase at
[anokye-labs.github.io/kbexplorer](https://anokye-labs.github.io/kbexplorer/)
has been rendering frontmatter-facts-as-structured-panel plus a live inline
Mermaid diagram from real, authored content since that merge. This satisfies
the acceptance criteria stated in
[kbexplorer#13](https://github.com/anokye-labs/kbexplorer/issues/13)'s
"Wave 0 done" section. **The rich-Markdown capability is shipped, released,
and live — not dormant.**

## The Copilot canvas epic

Two epics frame this work: [`kbexplorer-template#401`](https://github.com/anokye-labs/kbexplorer-template/issues/401)
("Copilot canvas representation target — host-themeable, embeddable",
de-risking/shared infra, **closed**) and
[`kbexplorer-template#407`](https://github.com/anokye-labs/kbexplorer-template/issues/407)
("bespoke copilot Representation target — agent-driven, anchor-first", the
destination surface, **still open** — 3 of 6 sub-issues shipped as of this
writing).

Three PRs landed in sequence on 2026-07-01, each unblocking the next:

| Merged (UTC) | PR | Closes issue | What |
|---|---|---|---|
| 01:14:05 | [template#441](https://github.com/anokye-labs/kbexplorer-template/pull/441) | [#406](https://github.com/anokye-labs/kbexplorer-template/issues/406) | Embeddable headless canvas mount (`canvas.html`/`canvas.tsx`) + `inherit-host` visual mode. Renders through the existing `spa` Representation — nothing new rendered yet, just a new host. |
| 01:44:18 | [template#442](https://github.com/anokye-labs/kbexplorer-template/pull/442) | [#440](https://github.com/anokye-labs/kbexplorer-template/issues/440) | Registers a distinct `copilot` Representation target. Initially delegates to the `spa` route tree — the seam, not yet the bespoke view. |
| 02:20:08 | [template#443](https://github.com/anokye-labs/kbexplorer-template/pull/443) | [#408](https://github.com/anokye-labs/kbexplorer-template/issues/408) | Replaces the `copilot` target's `spa` delegation with the **anchor-first** landing — the first bespoke (non-reused) piece of the canvas UX. The constellation is no longer the default view for this target. |

[`template#444`](https://github.com/anokye-labs/kbexplorer-template/pull/444)
*"chore(release): 0.4.0"* merged **02:33:28Z**, cutting the first template
release carrying the bespoke Copilot canvas surface. Per the CLI's
`getLatestTag()` (a dynamic `git ls-remote --tags` lookup at install/update
time — see
[`kbexplorer-cli/src/lib/version.js`](https://github.com/anokye-labs/kbexplorer-cli/blob/main/src/lib/version.js)),
this is not a hardcoded pin: any `kbx init`/`kbx update` run after this tag
picks up `v0.4.0` and the canvas surface automatically.

**Still open in template#407** (not yet built, as of this writing): agent
action surface (`anchor`/`expand`/`trace`/`filter` beyond the do-seam's raw
HTTP contract), bidirectional click→chat, an affordance-aware node launchpad,
and a fully conversation-shaped shell. See
[surfaces.md](surfaces.md#why-the-distinction-matters).

### The do-seam that the canvas calls into

In parallel, `kbexplorer-cli` built the loopback server the canvas talks to:
[`cli#199`](https://github.com/anokye-labs/kbexplorer-cli/pull/199) "A1:
Loopback canvas server + real `open()`" (closes
[#190](https://github.com/anokye-labs/kbexplorer-cli/issues/190)),
[`cli#200`](https://github.com/anokye-labs/kbexplorer-cli/pull/200) (`/manifest`
+ `/search`), and [`cli#201`](https://github.com/anokye-labs/kbexplorer-cli/pull/201)
(`/events` SSE + `/affordance/:name` do-seam). The affordance action contract
itself — the protocol-neutral core these all route through — was delivered
earlier by [`kbexplorer#21`](https://github.com/anokye-labs/kbexplorer/issues/21)
("Epic: Affordance action layer (the DO-seam), job layer and consent",
closed) via `kbexplorer-cli#153` (contract). See
[architecture.md](architecture.md#the-do-seam--affordances-as-a-protocol-neutral-action-layer)
for the adapter model this enables.

## Cross-source connection — "connect, not merge"

[`kbexplorer#15`](https://github.com/anokye-labs/kbexplorer/issues/15) ("Epic:
Cross-source connection (edge-minting, referent conflation, SoR-precedence)",
closed) shipped four `kbexplorer-cli` features, in this dependency order:
reference edge-minting (`cli#137`), referent conflation (`cli#138`),
SoR-precedence resolution (`cli#139`), and committed connection artifacts +
`--check` parity (`cli#140`). See
[decisions/conflation-correction.md](decisions/conflation-correction.md) for
the design reasoning this epic encodes.

## `kbexplorer-search`

A standalone semantic-search companion module
(`@anokye-labs/kbexplorer-search`), driven through the `kbx` CLI
(`kbx search-index`, `kbx search`). Shipped its package skeleton and core
search engine first, then graph-aware ranking, an optional FAISS-accelerated
index with graceful fallback, a runnable serve service matching the template's
search contract, and — most recently — access-label filtering so restricted or
unknown-access content is excluded from the index (E5-A3).

## Honest status, as of this writing

**Merged, released, and verified live end-to-end:**
- Mermaid rendering in `ReadingView` (template `v0.2.0`+).
- Rich-Markdown block rendering + authored ingestion, live on the showcase via
  `content/org/platform.md` (template `v0.3.0`+, this repo since
  `kbexplorer#95`).
- The `copilot` Representation target + embeddable canvas mount + anchor-first
  home view (template `v0.4.0`).
- The do-seam contract + extension-tool adapter + MCP adapter + canvas
  `/affordance/:name` adapter (`kbexplorer-cli`, `kbexplorer#21` closed).
- Cross-source connection primitives — edge-minting, referent conflation,
  SoR-precedence (`kbexplorer#15` closed) — though these are not yet exercised
  by any real multi-source (e.g. GitHub↔Azure DevOps) content in this repo's
  showcase; they are merged and tested in `kbexplorer-cli`, not yet
  demonstrated end-to-end here.

**Merged but narrow — real code, small surface:**
- The `copilot` target's anchor-first view is the *only* shipped piece of
  `template#407`'s six-item bespoke-canvas epic. Agent actions, click→chat,
  the node launchpad, and the conversation-shaped shell are designed
  (see the epic body) but not built.
- The canvas's `/events` SSE endpoint is live but its domain triggers
  (file-watch → `graph-updated`, canvas action → `anchor`) are still a no-op
  heartbeat-only seam — real triggers are an explicit follow-up.

**Designed but not built:**
- The forward-looking rendering-default question in
  [decisions/markdown-rendering-default.md](decisions/markdown-rendering-default.md)
  — whether display mode should be provider-scoped rather than a frontmatter
  opt-in gate, and whether embedded blocks should become first-class `kg://`
  nodes — is an open design question, not a decision, let alone shipped code.
