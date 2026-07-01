# Decision: connect sources, don't merge them

**Status: decided and shipped.** Grounded in
[`kbexplorer#15`](https://github.com/anokye-labs/kbexplorer/issues/15) (closed),
[`kbexplorer#86`](https://github.com/anokye-labs/kbexplorer/issues/86), and the
"Equivalence" section of
[`kbexplorer#12`](https://github.com/anokye-labs/kbexplorer/issues/12). No
single ADR document states this in one place across the org's history — this
page consolidates the decision from that issue cluster, quoting rather than
paraphrasing wherever it matters.

## The naive approach this rejects

The obvious way to federate "many sources → one graph" is fuzzy matching:
compare names, addresses, titles across sources, compute a confidence score,
and auto-merge above a threshold. kbx explicitly does not do this. From
`kbexplorer#15`:

> Connect — **not merge** — artifacts across sources... Three operations, no
> confidence-threshold auto-merge.

There is a real cost to rejecting the convenient version: no single pass
"solves" federation. But confidence-threshold auto-merge is exactly the kind
of silent, irreversible decision the rest of this system design tries to
avoid (see the [do-seam](../architecture.md#the-do-seam--affordances-as-a-protocol-neutral-action-layer)'s
fail-closed consent model, and `kbexplorer#12`'s "propose, don't overwrite"
principle) — it would let two sources' records collapse into one node with no
review step and no way back.

## What ships instead — three narrow, deterministic operations

`kbexplorer#15` scopes cross-source connection to exactly three operations,
each delivered as its own `kbexplorer-cli` feature (all closed, all merged —
see [history.md](../history.md#cross-source-connection--connect-not-merge)):

1. **Reference edge-minting between distinct artifacts**
   ([`cli#137`](https://github.com/anokye-labs/kbexplorer-cli/issues/137)) —
   the high-value case: a doc *describes* an epic, a PR *implements* a work
   item. The two artifacts stay two nodes; a typed edge connects them.
2. **Referent conflation of shared people/teams/services**
   ([`cli#138`](https://github.com/anokye-labs/kbexplorer-cli/issues/138)) —
   generalizes the existing `person.linked` mechanism: one node with multiple
   source-pointers, for the narrow case where the *same real-world entity* is
   genuinely named in more than one source (a person in a directory and in a
   calendar feed, say) — not a general similarity merge.
3. **SoR-precedence resolution for conflicting facts**
   ([`cli#139`](https://github.com/anokye-labs/kbexplorer-cli/issues/139)) —
   for the rare case where two sources disagree about the same fact, a
   **declared** precedence order wins, deterministically. No heuristic
   arbitration.

A fourth feature,
[`cli#140`](https://github.com/anokye-labs/kbexplorer-cli/issues/140),
requires the results to be **committed, deterministic git artifacts**
(`.kbx/connection/conflation-map.json`, `minted-edges.json`,
`manual-overrides.yaml`) with `kbx connect --check` parity in CI — so
conflation state is reviewable, diffable, and never silently regenerated
differently on two machines. Per `kbexplorer#15`:

> LLM suggestions land **only** as committed overrides/rules a human reviews.

## The corollary: there is no single canonical source

This design has a direct consequence worth stating explicitly: **kbx does not
assume, and does not need, one authoritative system of record.** Multiple
sources can describe overlapping reality; the graph's job is to connect their
witnesses (edges + conflation), not to pick a winner and discard the rest —
except in the narrow, declared-precedence case of a genuinely conflicting
fact. This matches `kbexplorer#12`'s framing of the underlying use case:

> The adopter is standing up an organizational knowledge graph... assembled by
> indexing the organization's authoritative source**s** [plural], never by
> rewriting them. The rule is "Index, don't migrate."

And the headline journey this whole epic serves,
[`kbexplorer#86`](https://github.com/anokye-labs/kbexplorer/issues/86)
("Morgan - multi-source unify"):

> many sources -> one graph by **connection, not merge** - mint reference
> edges between distinct artifacts, conflate shared people/teams/services,
> declare SoR precedence for rare conflicts.

## Where this sits in the sequencing — the adopter-driven re-center

`kbexplorer#13` (the foundation program epic) deliberately defers this work
rather than starting with it, even though "many sources → one graph" is the
program's headline. Its "Wave 0" scopes to **identity (E1) + rich-Markdown
rendering (E6) only**, stating explicitly that "none of it needs
GitHub/ADO API machinery," and lists cross-source connection (E3), decoupling
GitHub-the-host (E4), and access/governance labeling (E5) as **"Deferred epic
stubs (map only; children later)."** The execution plan
([`kbexplorer#90`](https://github.com/anokye-labs/kbexplorer/issues/90))
confirms the same order: "Wave 0a" is the core release gate, "Wave 0b" is the
rich-Markdown first-visible slice, and only later waves reach E3/E4/E5.

In other words: the adoption-facing, single-source-of-truth-per-doc slice
(stable identity + a document you can actually read) was sequenced ahead of
the harder, more architecturally interesting multi-source-unification slice
(this connect-not-merge design) — even though the latter is the more
compelling long-term story. That re-centering is sequencing evidence in
`kbexplorer#13`/`#90`, not a separately-written narrative document; it is
recorded here because it explains *why* `kbexplorer#15` shipped when it did
(after E1/E6, not before) rather than implying the connect-not-merge design
itself was an afterthought. See [history.md](../history.md) for the dated
account of when each wave actually shipped.
