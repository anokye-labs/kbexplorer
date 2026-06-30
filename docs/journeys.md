# Journeys

A journey is an **end-to-end narrative** for one [persona](personas.md) — the arc from
where they start to the value they reach. Journeys are how we pressure-test the system
as a whole: each one must be tellable without hand-waving, and each step must be
delivered by a tracked issue.

Where a journey says *delivered by*, it links the implementation issues that make the
step real. The demand-map umbrella is
[#23](https://github.com/anokye-labs/kbexplorer/issues/23); the two delivery programs
are the **foundation** ([#13](https://github.com/anokye-labs/kbexplorer/issues/13)) and
the **plugin** ([#8](https://github.com/anokye-labs/kbexplorer/issues/8)). See the
[roadmap](roadmap.md) for how these sequence.

| # | Journey | Persona | The crux |
|---|---|---|---|
| J1 | Greenfield genesis | Mei / Dana | Empty repo → living KB, with or without the CLI |
| J2 | Work items into the graph | Priya | A non-GitHub system of record is just a source |
| J3 | Perforce, no GitHub host | Kenji | Decouple the host; keep git as the backing store |
| J4 | The no-repo analyst | Noor | No git/Node/terminal — the plugin brokers everything |
| J5 | Many sources, one graph | Morgan | Connection, not merge (the headline) |
| J6 | Compliance & governance | Elena | Provenance, access, and approval that travel with the resource |
| J7 | The non-engineer | Quinn | Make what they already write addressable, time-aware, safe |

---

## J1 — Greenfield genesis
**Persona:** Mei / Dana · **From an empty repo to a living, queryable, visual KB —
with or without the CLI.**

Today the only door is `npx @anokye-labs/kbexplorer init`, and it has silent cliffs:
the visual-mode and theme you pick are discarded, semantic search is never mentioned,
and first-run failures (no Node 22, no git remote, vendor-mode `npm install` skip) are
undiscoverable. The plugin should **own the genesis** and turn those cliffs into
explicit choices.

Genesis is a **state machine, not a surface**: `template strategy → content mode →
visual identity → search mode → generate`, rendered by any surface. Three paths stay
distinct:

1. **CLI-first** — Dana runs the CLI; the canvas/MCP attach to the produced artifacts.
2. **Plugin-first** — Mei never touches the CLI: install + enable the plugin, open the
   onboarding canvas, and the plugin drives `init`/`generate`/`search-index` locally
   via the Copilot runtime. *(No CLI **for the user** — not zero-install.)*
3. **Read-only existing-KB** — point the canvas at a repo that already has a manifest.

**Delivered by:** [#20](https://github.com/anokye-labs/kbexplorer/issues/20) (genesis),
[#19](https://github.com/anokye-labs/kbexplorer/issues/19) (packaging);
[kbexplorer-cli#149](https://github.com/anokye-labs/kbexplorer-cli/issues/149) (state
machine), [#152](https://github.com/anokye-labs/kbexplorer-cli/issues/152) (first-run);
[kbexplorer-template#428](https://github.com/anokye-labs/kbexplorer-template/issues/428)
(canvas onboarding).

---

## J2 — Priya: Azure DevOps work items into the graph
**Persona:** Priya · **A non-GitHub system of record is just another source.**

Priya maps work items, epics, iterations, risks, and ownership from Azure DevOps. ADO
is ingested through a **generic provider**; identity is source-agnostic (no GitHub
shape assumed); references carried in ADO content (`AB#1234`, links to repos/PRs)
resolve to target nodes and become **minted edges**; access labels carry over.

**Delivered by:** [kbexplorer-core#18](https://github.com/anokye-labs/kbexplorer-core/issues/18)
(identity); [kbexplorer-cli#132](https://github.com/anokye-labs/kbexplorer-cli/issues/132),
[#135](https://github.com/anokye-labs/kbexplorer-cli/issues/135) (ingestion);
[#15](https://github.com/anokye-labs/kbexplorer/issues/15) (connection);
[#17](https://github.com/anokye-labs/kbexplorer/issues/17) (access).

---

## J3 — Kenji: Perforce Helix Core (no GitHub host)
**Persona:** Kenji · **Decouple the host; keep git as the backing store.**

Kenji works in Perforce — changelists, streams, assets — with no GitHub host. kbx must
separate *GitHub-the-host* (gh, owner/repo identity, Pages deploy, PR handoff) from the
engine, while keeping **git** as the optional store. "Affected" becomes a
changelist/shelf/label range rather than a git diff; the deepest coupling is the
change-proposal (PR) handoff.

**Delivered by:** [#16](https://github.com/anokye-labs/kbexplorer/issues/16) (decouple
host: [kbexplorer-cli#141](https://github.com/anokye-labs/kbexplorer-cli/issues/141),
[#142](https://github.com/anokye-labs/kbexplorer-cli/issues/142));
[kbexplorer-cli#135](https://github.com/anokye-labs/kbexplorer-cli/issues/135) (provider),
[#136](https://github.com/anokye-labs/kbexplorer-cli/issues/136) (affected dispatch).

---

## J4 — Noor: the no-repo analyst
**Persona:** Noor · **No git, no Node, no terminal — the plugin brokers everything.**

Noor imports from Confluence / Notion / Drive and wants queryable org context but has
none of the developer scaffolding. The plugin must broker a **plugin-first genesis**; a
**GraphStore** (not a committed repo) holds state; non-GitHub sources are first-class.
This is the strongest test of "usable without the CLI."

**Delivered by:** [#20](https://github.com/anokye-labs/kbexplorer/issues/20)
(plugin-first genesis);
[kbexplorer-cli#135](https://github.com/anokye-labs/kbexplorer-cli/issues/135) (provider);
[kbexplorer-template#382](https://github.com/anokye-labs/kbexplorer-template/issues/382)
(GraphStore); [#16](https://github.com/anokye-labs/kbexplorer/issues/16) (host-decouple).

---

## J5 — Morgan: many sources, one graph (the headline)
**Persona:** Morgan · **Connection, not merge.**

Morgan maintains teams, systems, owners, missions, and decisions across **multiple**
sources. The headline value is *many sources → one graph* — but not by merging
duplicate records. Three operations: **mint reference edges** between distinct
artifacts (a Confluence doc *describes* a GitHub epic; a PR *implements* an ADO work
item); **conflate referents** (the same person/team/service under several native IDs
collapses to one node with multiple source-pointers); and **declare source precedence**
for the rare conflicting fact. No confidence-threshold auto-merge.

**Delivered by:** [#15](https://github.com/anokye-labs/kbexplorer/issues/15)
(connection); [kbexplorer-core#18](https://github.com/anokye-labs/kbexplorer-core/issues/18)
(identity); [kbexplorer-cli#132](https://github.com/anokye-labs/kbexplorer-cli/issues/132)
(ingestion).

---

## J6 — Elena: compliance & governance
**Persona:** Elena · **Trust that travels with the resource.**

Elena audits provenance, stale decisions, approvals, and access boundaries across a
unified KB. She needs first-class **provenance** (observed vs derived, sources,
generator), **access labels** that travel with the resource (kbx labels; the host
enforces), a **human approval gate** for generated content, and **consent + cost
disclosure** for the sampling bridge.

**Delivered by:** [#17](https://github.com/anokye-labs/kbexplorer/issues/17) (access);
[kbexplorer-core#23](https://github.com/anokye-labs/kbexplorer-core/issues/23),
[#24](https://github.com/anokye-labs/kbexplorer-core/issues/24) (provenance/derivation);
[kbexplorer-cli#155](https://github.com/anokye-labs/kbexplorer-cli/issues/155) (consent);
[kbexplorer-template#429](https://github.com/anokye-labs/kbexplorer-template/issues/429)
(review).

---

## J7 — Quinn: the non-engineer knowledge worker
**Persona:** Quinn · **Meet them where they are.**

Quinn lives in a spreadsheet, builds the deck, iterates the doc. A resource-oriented
model — articulated in
[#12's discussion](https://github.com/anokye-labs/kbexplorer/issues/12) — makes the
structured part of what Quinn already writes:

- **addressable** — one thing, one address; many views. Stop hand-reconciling the doc,
  the sheet, and the slide as three drifting copies.
- **navigable** — follow meaning, not folders, because the links *are* the structure.
- **time-aware** — the system remembers *when*, not just *what* ("what did we commit to
  in C1" vs "what did we believe at the time").
- **safe to change** — a change is a reviewable **proposal** routed to the right
  authority, not a silent clobber; the system tells you in the moment what you may
  change, and the data explains its own vocabulary and constraints.

Judgment stays human; plumbing goes to agents that can only do what a resource
advertises.

**Delivered by:** [kbexplorer-core#18](https://github.com/anokye-labs/kbexplorer-core/issues/18)
(identity); content negotiation (story D9) and advertised-operations safety (story E8)
in the [demand map](user-stories.md); the **Time & provenance** family (K); propose/review
via [kbexplorer-cli#142](https://github.com/anokye-labs/kbexplorer-cli/issues/142) +
[kbexplorer-template#429](https://github.com/anokye-labs/kbexplorer-template/issues/429).
