# Hosting a kbx dataset

How a dataset goes from a git repository to a served knowledge base that
people and agents consume. The crux: **GitHub-the-host is separable from
git-the-store**. The dataset lives in git; GitHub Pages is the convenient
default host for its rendered representations — but a paved alternative, not a
dependency.

![Hosting a kbx dataset — one repository, many representations, served as a static site](hosting-a-dataset.svg)

## The flow

1. **The dataset lives in git** — authored content, manifest, `.kbx.json`,
   and the committed search index, versioned together. The repository *is*
   the system of record for the overlay itself.
2. **Build on merge** — a GitHub Actions workflow runs `generate` when a
   change lands on `main`. The build is deterministic: same commit, same
   output.
3. **Project representations** — the engine's pure `KBGraph` is projected
   through the representation layer: `spa` for humans, `json-ld` for interop,
   `llm-context` for agents. One graph, many targets — see
   [architecture, layer 4](../architecture.md#layer-4--representation).
4. **Deploy the static site** — the rendered output ships to GitHub Pages or
   any static host. Host-decoupling keeps `gh`-specific concerns (owner/repo
   identity, Pages deploy, PR handoff) out of the engine.
5. **Consume** — people browse, search, and query through the SPA and canvas;
   agents load the `llm-context` pack or call the MCP tools over the same
   graph.

## Gates & guarantees

- **Run modes are conceptually equivalent.** Local manifest mode and remote
  runtime mode may differ in transport, but they must not silently produce
  different knowledge domains.
- **Representations are projections.** The SPA, an embeddable canvas, and an
  agent surface consume the same pure graph rather than forking the system.
- **The host is swappable.** A dataset hosted off-GitHub keeps git as its
  store; only the host adapter changes.

## Traceability

- Stories: [G1 — install & distribute](../user-stories.md#g--operate--30),
  [G5 — run surfaces with one lifecycle](../user-stories.md#g--operate--30),
  [D9 — content negotiation](../user-stories.md#d--make-sense-of-it--27),
  [F1/F2 — llm-context & JSON-LD export](../user-stories.md#f--kb-as-context--export--29).
- Journey: [J3 — no GitHub host](../journeys.md#j3--kenji-perforce-helix-core-no-github-host)
  is the strongest test of the host seam.
- Delivered by: [#16](https://github.com/anokye-labs/kbexplorer/issues/16) (decouple host),
  [kbexplorer-cli#131](https://github.com/anokye-labs/kbexplorer-cli/issues/131) (lifecycle),
  [kbexplorer-cli#92](https://github.com/anokye-labs/kbexplorer-cli/issues/92) /
  [#93](https://github.com/anokye-labs/kbexplorer-cli/issues/93) (representation targets),
  [kbexplorer-template#423](https://github.com/anokye-labs/kbexplorer-template/issues/423) (surfaces).
