# User stories — the demand map

This is the **demand side** of kbx: what people need, expressed as user stories and
grouped into families. The **supply side** — the work that delivers them — lives in the
two programs (**foundation**
[#13](https://github.com/anokye-labs/kbexplorer/issues/13) and **plugin**
[#8](https://github.com/anokye-labs/kbexplorer/issues/8)) and is sequenced by the
[roadmap](roadmap.md). Every story below links both to its **tracking issue** (under
the demand-map umbrella [#23](https://github.com/anokye-labs/kbexplorer/issues/23)) and
to the issue(s) that **deliver** it, so coverage is traceable in both directions.

Personas are introduced in [personas.md](personas.md); end-to-end arcs in
[journeys.md](journeys.md).

**Locus** legend — *where the capability mainly lives*: **C** = core contract ·
**S** = shared interaction layer · **U** = surface UX. Locus is not cost: the rendering
seam is cheap; genesis, write-back, the affordance launchpad, and the credential/agent
plumbing are the real investment.

---

## A — Genesis (stand up a KB) · [#24](https://github.com/anokye-labs/kbexplorer/issues/24)

- **A1 — Cold-start guided init** ([#34](https://github.com/anokye-labs/kbexplorer/issues/34)) · *As Mei/Dana, I want a guided cold-start from an empty repo, so that I reach a living, queryable KB without the terminal.* · **U** · delivers: #20, cli#149/#152, template#428.
- **A2 — Choose template strategy** ([#35](https://github.com/anokye-labs/kbexplorer/issues/35)) · *…choose submodule/vendor/custom/ref, recorded in `.kbx.json`, so that I control the upgrade path.* · **C** · delivers: cli#149.
- **A3 — Persist visual identity + theme** ([#36](https://github.com/anokye-labs/kbexplorer/issues/36)) · *…the visual-mode + theme I pick to persist, so that my KB looks the way I chose.* · **C/bugfix** · delivers: cli#150, core#15.

## B — Grow & curate · [#25](https://github.com/anokye-labs/kbexplorer/issues/25)

- **B1 — Add a node by hand** ([#37](https://github.com/anokye-labs/kbexplorer/issues/37)) · **S** · delivers: cli#133, template#427.
- **B2 — Incidental node from a code change** ([#38](https://github.com/anokye-labs/kbexplorer/issues/38)) · **C** · delivers: cli#136/#158.
- **B3 — Connect two nodes with a typed relation** ([#39](https://github.com/anokye-labs/kbexplorer/issues/39)) · **S/C** · delivers: core#27.
- **B4 — New node type + bespoke viewer** ([#40](https://github.com/anokye-labs/kbexplorer/issues/40)) · **C+U** · delivers: template#147, cli#148.
- **B5 — Curate clusters & structure** ([#41](https://github.com/anokye-labs/kbexplorer/issues/41)) · **S** · delivers: template#147/#54.
- **B6 — Generate content for a stub via an agent** ([#42](https://github.com/anokye-labs/kbexplorer/issues/42)) · **C** · delivers: cli#147, template#429.

## C — Keep healthy & current · [#26](https://github.com/anokye-labs/kbexplorer/issues/26)

- **C1 — Incremental refresh after a code change** ([#43](https://github.com/anokye-labs/kbexplorer/issues/43)) · **C** · delivers: cli#136/#158.
- **C2 — Freshness & staleness signals** ([#44](https://github.com/anokye-labs/kbexplorer/issues/44)) · **S** · delivers: cli#157.
- **C3 — CI gates drift** ([#45](https://github.com/anokye-labs/kbexplorer/issues/45)) · **C** · delivers: cli#140/#151.
- **C4 — Audit structural integrity** ([#46](https://github.com/anokye-labs/kbexplorer/issues/46)) · **C** · delivers: core#18.
- **C5 — Update template version safely** ([#47](https://github.com/anokye-labs/kbexplorer/issues/47)) · **C** · delivers: cli#42/#117.
- **C6 — Validate a proposed change against a shape** ([#74](https://github.com/anokye-labs/kbexplorer/issues/74)) · *…against a declared SHACL/ShEx shape before commit, so that invalid data never lands and the data explains its own constraints.* · **C/S** · delivers: new validation work; template#429. *(from [#12](https://github.com/anokye-labs/kbexplorer/issues/12) comments)*

## D — Make sense of it · [#27](https://github.com/anokye-labs/kbexplorer/issues/27)

- **D1 — Explore the constellation** ([#48](https://github.com/anokye-labs/kbexplorer/issues/48)) · **U** · delivers: template#401/#408.
- **D2 — Focused reading view** ([#49](https://github.com/anokye-labs/kbexplorer/issues/49)) · **U** · delivers: template#426/#412.
- **D3 — Semantic, graph-aware search** ([#50](https://github.com/anokye-labs/kbexplorer/issues/50)) · **S** · delivers: template#344, cli#151.
- **D4 — Follow relations hop-by-hop** ([#51](https://github.com/anokye-labs/kbexplorer/issues/51)) · **S** · delivers: template#409.
- **D5 — Copilot answers grounded in the KB** ([#52](https://github.com/anokye-labs/kbexplorer/issues/52)) · **S** · delivers: cli#153.
- **D6 — Node-type-specific viewer** ([#53](https://github.com/anokye-labs/kbexplorer/issues/53)) · **U** · delivers: template#412/#147.
- **D7 — Cross-cutting question across clusters** ([#54](https://github.com/anokye-labs/kbexplorer/issues/54)) · **S** · delivers: template#344, cli#153.
- **D8 — Home/anchor orientation view** ([#55](https://github.com/anokye-labs/kbexplorer/issues/55)) · **U** · delivers: template#408.
- **D9 — Request a resource in a chosen representation** ([#75](https://github.com/anokye-labs/kbexplorer/issues/75)) · *…via content negotiation (spa/json-ld/llm-context/canvas), so that one resource serves many views.* · **S** · delivers: cli#92/#93, core#16. *(from #12 comments)*

## E — Affordance-aware actions on any node · [#28](https://github.com/anokye-labs/kbexplorer/issues/28)

> "Act on it" is **affordance-generic**, not issue/PR. Issue/PR are one instance of an
> open affordance model — never the frame.

- **E1 — See affordances on any node** ([#56](https://github.com/anokye-labs/kbexplorer/issues/56)) · **C/S** · delivers: template#411.
- **E2 — Edit a content node + stage write-back** ([#57](https://github.com/anokye-labs/kbexplorer/issues/57)) · **C** · delivers: cli#142.
- **E3 — Act on a work node (one affordance instance)** ([#58](https://github.com/anokye-labs/kbexplorer/issues/58)) · **C** · delivers: cli#142.
- **E4 — Derive nodes from an arbitrary source** ([#59](https://github.com/anokye-labs/kbexplorer/issues/59)) · **C** · delivers: cli#135.
- **E5 — Agent acts via the canvas click→chat bridge** ([#60](https://github.com/anokye-labs/kbexplorer/issues/60)) · **U** · delivers: template#410.
- **E6 — Trigger an agent task from a node** ([#61](https://github.com/anokye-labs/kbexplorer/issues/61)) · **S** · delivers: template#409, cli#154.
- **E7 — Scaffold new structure from a node** ([#62](https://github.com/anokye-labs/kbexplorer/issues/62)) · **C** · delivers: cli#146.
- **E8 — Agent acts only on advertised operations** ([#76](https://github.com/anokye-labs/kbexplorer/issues/76)) · *…in-band (HATEOAS/Hydra), so that handing work to an agent is trustworthy by construction.* · **C/S** · delivers: template#411, cli#155. *(from #12 comments)*

## F — KB as context & export · [#29](https://github.com/anokye-labs/kbexplorer/issues/29)

- **F1 — Token-budgeted llm-context pack** ([#63](https://github.com/anokye-labs/kbexplorer/issues/63)) · **C** · delivers: cli#92/#153.
- **F2 — Export JSON-LD for interop** ([#64](https://github.com/anokye-labs/kbexplorer/issues/64)) · **C** · delivers: cli#92/#93.
- **F3 — Expose the KB as MCP tools** ([#65](https://github.com/anokye-labs/kbexplorer/issues/65)) · **C** · delivers: cli#153.

## G — Operate · [#30](https://github.com/anokye-labs/kbexplorer/issues/30)

- **G1 — Install & distribute the plugin** ([#66](https://github.com/anokye-labs/kbexplorer/issues/66)) · **U** · delivers: cli#145.
- **G2 — Wire MCP config + credentials** ([#67](https://github.com/anokye-labs/kbexplorer/issues/67)) · **U/ops** · delivers: cli#156.
- **G3 — doctor / preflight verifies setup** ([#68](https://github.com/anokye-labs/kbexplorer/issues/68)) · **C** · delivers: cli#152/#100.
- **G4 — Opt into search; build + commit index** ([#69](https://github.com/anokye-labs/kbexplorer/issues/69)) · **C** · delivers: cli#151, template#344.
- **G5 — Run surfaces with one lifecycle** ([#70](https://github.com/anokye-labs/kbexplorer/issues/70)) · **U/ops** · delivers: cli#131, template#423.

## H — Sync & trust · [#31](https://github.com/anokye-labs/kbexplorer/issues/31)

- **H1 — Stay in sync with the source (drift loop)** ([#71](https://github.com/anokye-labs/kbexplorer/issues/71)) · **S/C** · delivers: cli#157/#158/#159.
- **H2 — Review generated content before publish** ([#72](https://github.com/anokye-labs/kbexplorer/issues/72)) · **C/S/U** · delivers: template#429, cli#158/#155.
- **H3 — Multi-repo / org-level genesis** ([#73](https://github.com/anokye-labs/kbexplorer/issues/73)) · **S** · delivers: cli#10/#11, #20.

## K — Time & provenance (semantic space-time) · [#32](https://github.com/anokye-labs/kbexplorer/issues/32)

> From [#12](https://github.com/anokye-labs/kbexplorer/issues/12)'s discussion: a
> bitemporal + PROV-O model. **K1–K3 are new core work** not yet in E1–E6 — a candidate
> new epic; these stories are the demand signal for it.

- **K1 — Bitemporal facts (valid time vs recorded time)** ([#77](https://github.com/anokye-labs/kbexplorer/issues/77)) · *…so that future-valid and backdated facts are ordinary, not special cases.* · **C (new)** · aligns core#24.
- **K2 — As-of queries (time travel)** ([#78](https://github.com/anokye-labs/kbexplorer/issues/78)) · *…what we believed/committed as of time t, so that planning and audit stop being archaeology.* · **C/S (new)**.
- **K3 — Future-valid & backdated facts** ([#79](https://github.com/anokye-labs/kbexplorer/issues/79)) · **C (new)**.
- **K4 — Honest derivation that propagates** ([#80](https://github.com/anokye-labs/kbexplorer/issues/80)) · *…along `wasDerivedFrom`, flagging stale dependents without deleting history.* · **C** · delivers: core#24, cli#157/#158.
- **K5 — Provenance & citations on every fact** ([#81](https://github.com/anokye-labs/kbexplorer/issues/81)) · **C** · delivers: core#23/#24.

---

## Journeys

The end-to-end persona narratives (J1–J7) are tracked under
[#33](https://github.com/anokye-labs/kbexplorer/issues/33) and told in
[journeys.md](journeys.md).
