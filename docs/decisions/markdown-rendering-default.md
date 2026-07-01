# Open question: should rich rendering be the default, not an opt-in?

**Status: NOT DECIDED. NOT BUILT.** This page records a design tension raised
during adoption planning. It is deliberately written as an open question with
tradeoffs, not a decision — the granularity and identity questions below are
genuinely unresolved. Do not treat anything here as a commitment.

## The status quo being reconsidered

Today, a Markdown document only gets kbx's richest rendering — frontmatter
lifted into a structured facts panel, embedded `mermaid`/`dot`/`ics`/`canvas`
blocks rendered live or via pre-built-SVG fallback — if it explicitly opts in
with `display: rich-markdown` in its YAML frontmatter. The gate is a single,
narrow discriminator function:

```ts
// kbexplorer-template/src/engine/providers/rich-markdown/detect.ts
export function isRichAuthoredMarkdown(raw: string): boolean {
  return readFrontmatterDisplay(raw) === 'rich-markdown';
}
```

This was a deliberate choice, not an oversight — the same file's comment
explains why: an explicit flag (rather than auto-detecting fenced blocks)
"leaves existing authored docs that merely embed a Mermaid diagram... render
via the plain prose path — completely untouched, with no change to their id,
identity, or display." It protects existing content from an unannounced
identity/rendering change. See [history.md](../history.md) for when this
shipped ([`template#432`](https://github.com/anokye-labs/kbexplorer-template/pull/432)).

## The tension

The concern raised: **almost nobody hand-adds frontmatter to a bulk-imported
Markdown repository.** If the goal is "index, don't migrate" — pull in an
org's existing Markdown corpus as-is — an opt-in frontmatter gate means the
richest rendering never fires on real, pre-existing content unless someone
goes back and edits every file. That inverts the adoption story: the feature
exists but the content that would benefit from it structurally cannot reach
it without a migration step the whole design is trying to avoid.

The suggested alternative direction has three parts, in increasing order of
how unresolved they are.

### (a) All markdown should render with the best available renderer by default

Rather than requiring an explicit opt-in per document, the default should be:
render every ingested Markdown document as richly as the available renderers
allow, and let a document opt **down** to raw rendering only when it actually
needs to (e.g. `display-mode: raw`) — inverting today's opt-in gate to an
opt-out escape hatch.

### (b) Display mode as a property of the source/provider, not the document

Rather than a per-document frontmatter flag, the render/display mode could be
a configuration property of the **Source** or **Provider** that ingested the
document — meaning two byte-identical `.md` files could render differently
depending on which provider brought them in (one repo's docs get rich
rendering by provider config; another repo's byte-identical copy, ingested via
a different provider, renders plain). This composes naturally with the
existing `Source`/`Provider` seams described in
[architecture.md](../architecture.md) — display mode would be exactly the
kind of per-source configuration those seams are built to carry — but no
prototype of this exists.

### (c) Embedded blocks as first-class `kg://` graph nodes

Today an embedded block (a fenced `mermaid`/`dot`/`ics`/`canvas` block inside a
rich-Markdown doc) is a part of its parent document's node — it has no
identity or edges of its own. The idea raised: some embedded blocks are
substantial enough to deserve **promotion** to their own graph node, with
their own `kg://` identity and edges back to the parent document. Worked
example: a `squads.md` document's frontmatter becomes a `squad` node's
structured data (as today), but an embedded block that references a linked
resource — say, a team roster or an org-chart fragment — could become its
own promotable node other parts of the graph can point at directly, rather
than only being reachable by first navigating to the parent document.

## Why this is not a decision yet

Each part above raises a genuinely open question with real tradeoffs, not
just an implementation detail:

- **(a) breaks the explicit protection** the current gate was built for —
  existing docs that merely embed a Mermaid diagram would change identity/
  display without anyone asking for it, unless the opt-down default is
  chosen very carefully (e.g. scoped only to *newly ingested* sources, not
  retroactively to existing ones).
- **(b) has no prototype.** It's a plausible extension of the existing
  `Source`/`Provider` config surface, but nobody has built or tested a
  provider-scoped display-mode config, and it isn't clear yet whether
  "display mode" belongs on the `Source` (which resource is retrieved) or the
  `Provider` (how it's turned into nodes) — they are different seams
  ([architecture.md](../architecture.md#layer-1--sources) vs
  [Layer 2](../architecture.md#layer-2--providers)).
- **(c) has two open, unresolved sub-questions of its own:**
  - **Granularity** — which embedded blocks are worth promoting? Every fenced
    block, or only ones that reference another resource? Promoting
    everything risks graph noise; promoting selectively requires a rule
    nobody has written yet.
  - **Identity scheme for promoted blocks** — a deterministic
    `kg://parent#block-index` (stable as long as block order doesn't change,
    but liable to drift if blocks are reordered or inserted) versus a
    content-hash-based identity (stable under reordering, but changes if the
    block's content changes at all, which may or may not be desirable
    depending on whether you want the promoted node to track its source
    block's edits). Neither has been chosen; both have precedent elsewhere
    in the system — `kg://` identity is content-independent by design
    ([architecture.md](../architecture.md#cross-cutting--identity--relations)),
    while the rich-Markdown block registry already computes a `contentHash`
    per block for change detection, so a hash-based option would reuse
    existing plumbing rather than invent new plumbing.

## What would need to happen before this becomes a decision

At minimum: a prototype of provider-scoped display-mode config against a
second real Source/Provider (to test whether the seam actually wants this
property), and a concrete proposal — with tradeoffs written down, not just
asserted — for the block-promotion granularity rule and identity scheme. Until
then, this stays exactly what it is: a documented question, not a plan.
