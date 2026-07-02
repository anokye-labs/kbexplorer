# Updating the search corpus

How the search corpus stays in lockstep with the dataset. The crux: **the
corpus is versioned with the graph and gated by CI** — search results are only
ever as fresh as the last green build, and that is a checkable property, not a
hope.

![Updating the search corpus — search is versioned with the graph](updating-the-search-corpus.svg)

## The flow

1. **Opt in to search** — search mode is one of the explicit genesis choices
   (see [creating a dataset](creating-a-dataset.md)) and can be enabled later.
   A dataset without search opt-in has no corpus and no index-freshness gate.
2. **Content lands on `main`** — a merged PR changes nodes, edges, or
   authored pages.
3. **Build the index** — the affected content is chunked and embedded into
   the corpus artifacts. Like [regeneration](updating-a-dataset.md), index
   builds are scoped to what changed.
4. **Commit the corpus** — the index is committed alongside the content it
   indexes, so a checkout is self-contained and reproducible. CI checks index
   freshness: a corpus that lags its content is a red build, not a mystery.
5. **Surfaces load the corpus** — the SPA, the canvas, and the MCP tools all
   serve semantic, graph-aware search from the same committed artifacts.
6. **Query** — results resolve to graph nodes with stable `kg://` identity,
   so a hit is a node you can open, traverse from, and act on — not a bare
   text snippet.

## Gates & guarantees

- **Same commit, same corpus.** The index and the content it describes move
  together; there is no out-of-band index deployment to drift.
- **Freshness is a check.** The gate in
  [rulesets & automation](rulesets-and-automation.md) fails a PR whose
  content changes are not reflected in the index.
- **Search is graph-aware.** Results carry node identity and relations, which
  is what lets agents ground answers rather than quote fragments.

## Traceability

- Stories: [G4 — opt into search; build + commit index](../user-stories.md#g--operate--30),
  [D3 — semantic, graph-aware search](../user-stories.md#d--make-sense-of-it--27),
  [D7 — cross-cutting questions](../user-stories.md#d--make-sense-of-it--27).
- Delivered by: [kbexplorer-cli#151](https://github.com/anokye-labs/kbexplorer-cli/issues/151)
  (index build + freshness), [kbexplorer-template#344](https://github.com/anokye-labs/kbexplorer-template/issues/344)
  (search surface), [kbexplorer-search](https://github.com/anokye-labs/kbexplorer-search)
  (the search package).
