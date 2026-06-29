# KBX adoption paved path

This page turns a recent failed attempt to incorporate kbx into another
repository into a product roadmap. The immediate technical gaps matter, but the
larger problem is that adoption currently depends on knowing implementation
details that should be guided, validated, or invisible.

The goal: a repository owner should be able to initialize kbx, point it at their
content, see the same conceptual graph in local and remote modes, trust that
typed relationships are preserved, render authored knowledge faithfully, and get
actionable diagnostics when the setup drifts.

For system contracts, link to
[`kbexplorer-core`](https://github.com/anokye-labs/kbexplorer-core) as the
source of truth. This document describes the user journey and owning work, not a
replacement contract.

## Adoption principle

Adoption should be a guided path, not an archaeology exercise:

1. **Make repository layout explicit.** If structured knowledge can live in more
   than one place, the path belongs in config and diagnostics, not in a hidden
   hardcoded default.
2. **Keep run modes conceptually equivalent.** Local manifest mode and remote
   runtime mode can differ in transport, but they should not silently produce
   different knowledge domains.
3. **Preserve authored semantics.** Typed relationships, direction, parallel
   edges, and diagrams are not decoration; they are the knowledge base.
4. **Surface capabilities before failure.** Protocol, template, provider, and
   optional-feature mismatches should be caught by compatibility checks and
   `doctor`, not discovered through a blank or partial UI.
5. **Treat UX surfaces as representations.** The SPA, an embeddable canvas, and
   a future conversation-anchored Copilot surface should consume the same pure
   graph rather than fork the system.

## Journey map

| Adoption stage | Desired experience | Current leak | Needed capability | Owning work |
|---|---|---|---|---|
| Initialize | Pick a template/version and know it works with the installed CLI. | CLI and template evolve independently; drift can fail silently. | Protocol version plus capability checks. | [`kbexplorer-cli#42`](https://github.com/anokye-labs/kbexplorer-cli/issues/42) |
| Choose repo layout | Put structured descriptors where the repo expects them. | The build only sees a top-level `content-model/` directory. | Configurable structured-content path with validation. | [`kbexplorer-template#416`](https://github.com/anokye-labs/kbexplorer-template/issues/416), [`kbexplorer-cli#129`](https://github.com/anokye-labs/kbexplorer-cli/issues/129) |
| Model structured content | Add typed entities and relationships without editing template internals. | The structured-content model is partly path/default driven and not yet fully open. | Open, data-driven node-type and provider model. | [`kbexplorer-template#147`](https://github.com/anokye-labs/kbexplorer-template/issues/147), [`kbexplorer-core#9`](https://github.com/anokye-labs/kbexplorer-core/issues/9) |
| Build locally | Generate a manifest that includes authored docs, repo activity, files, and structured descriptors. | Alternate structured-content paths are invisible. | Manifest generation should honor config and report missing inputs. | [`kbexplorer-template#416`](https://github.com/anokye-labs/kbexplorer-template/issues/416) |
| Run remotely | Runtime mode shows the same conceptual graph as local mode. | Remote mode registers the content-model provider as a no-op. | Remote structured-content fetch path through the source/host API. | [`kbexplorer-template#265`](https://github.com/anokye-labs/kbexplorer-template/issues/265) |
| Render authored knowledge | Markdown primitives render as meaningful documentation. | Fenced `diagram` blocks show a placeholder plus raw source. | Real diagram rendering with visible fallback on invalid input. | [`kbexplorer-template#417`](https://github.com/anokye-labs/kbexplorer-template/issues/417) |
| Preserve graph meaning | Multiple typed or directional relationships between two nodes survive graph build. | Edge deduplication collapses relationships by unordered endpoint pair. | Edge identity must include relation and direction where meaningful. | [`kbexplorer-template#418`](https://github.com/anokye-labs/kbexplorer-template/issues/418) |
| Embed and act | Host environments can render a useful kbx surface without reimplementing the graph. | The full SPA is optimized for human full-screen exploration, not narrow or agent-driven contexts. | Host-themeable canvas target and bespoke Copilot representation. | [`kbexplorer-template#401`](https://github.com/anokye-labs/kbexplorer-template/issues/401), [`kbexplorer-template#407`](https://github.com/anokye-labs/kbexplorer-template/issues/407), [`kbexplorer-core#15`](https://github.com/anokye-labs/kbexplorer-core/issues/15), [`kbexplorer-core#16`](https://github.com/anokye-labs/kbexplorer-core/issues/16) |
| Troubleshoot | One command explains whether the repo is ready and what is missing. | Users must infer setup errors from partial graphs or rendering gaps. | `kbx doctor` adoption-readiness diagnostics. | [`kbexplorer-cli#129`](https://github.com/anokye-labs/kbexplorer-cli/issues/129), [`kbexplorer-cli#100`](https://github.com/anokye-labs/kbexplorer-cli/issues/100) |

## What the four gaps reveal

### 1. Structured-content path is an adoption setting

The fixed `content-model/` directory is not just a missing parameter. It makes
repository layout a secret contract. A paved path needs a documented config
field, a backwards-compatible default, and a diagnostic that tells users when
structured content is present but not being loaded.

This belongs primarily in the template because manifest generation and loading
live there today, with CLI support so `init`, `update`, and `doctor` can explain
the setting.

### 2. Diagrams are part of authoring fidelity

A diagram fence that routes to a placeholder tells adopters that authored
knowledge is only partially supported. Rendering should be explicit about the
supported diagram language, render it in both run modes, and keep raw source
visible when rendering fails.

This is a representation concern in the template. It should not affect the core
graph contract.

### 3. Edge identity must preserve semantics

Typed structured knowledge often needs more than one relationship between the
same pair of entities. Collapsing edges by unordered endpoint pair erases
relation, type, and direction after providers have already expressed them.

The fix should preserve deterministic graph output while changing the
deduplication key to reflect the semantics already present in graph data. Any
golden diffs should be reviewable as intentional graph-correctness changes.

### 4. Local and remote mode need parity

Remote/runtime mode currently omits structured descriptor nodes because it does
not fetch and populate the content-model provider. From an adopter's
perspective, this is a mode-parity failure: the same repository should not
produce a different conceptual graph solely because it is loaded through the
host API instead of a prebuilt manifest.

The template issue already tracks the loader work. The broader adoption path
also needs diagnostics that explain this limitation while it exists.

## Issue matrix

| Area | Issue | Status |
|---|---|---|
| Docs adoption guide | [`kbexplorer#9`](https://github.com/anokye-labs/kbexplorer/issues/9) | Filed for this docs artifact |
| Enterprise reuse and template customization | [`kbexplorer-cli#10`](https://github.com/anokye-labs/kbexplorer-cli/issues/10) | Existing |
| CLI/template protocol checks | [`kbexplorer-cli#42`](https://github.com/anokye-labs/kbexplorer-cli/issues/42) | Existing |
| Adoption-readiness diagnostics | [`kbexplorer-cli#129`](https://github.com/anokye-labs/kbexplorer-cli/issues/129) | Filed |
| Search diagnostics precedent | [`kbexplorer-cli#100`](https://github.com/anokye-labs/kbexplorer-cli/issues/100) | Existing |
| Provider factory contract | [`kbexplorer-core#9`](https://github.com/anokye-labs/kbexplorer-core/issues/9) | Existing |
| Presentation-token contract | [`kbexplorer-core#15`](https://github.com/anokye-labs/kbexplorer-core/issues/15) | Existing |
| Canvas representation convention | [`kbexplorer-core#16`](https://github.com/anokye-labs/kbexplorer-core/issues/16) | Existing |
| Data-driven node-type engine | [`kbexplorer-template#147`](https://github.com/anokye-labs/kbexplorer-template/issues/147) | Existing |
| Remote structured-content ingestion | [`kbexplorer-template#265`](https://github.com/anokye-labs/kbexplorer-template/issues/265) | Existing, cross-linked |
| Configurable structured-content path | [`kbexplorer-template#416`](https://github.com/anokye-labs/kbexplorer-template/issues/416) | Filed |
| Diagram block rendering | [`kbexplorer-template#417`](https://github.com/anokye-labs/kbexplorer-template/issues/417) | Filed |
| Typed/directional parallel edges | [`kbexplorer-template#418`](https://github.com/anokye-labs/kbexplorer-template/issues/418) | Filed |
| Embeddable canvas target | [`kbexplorer-template#401`](https://github.com/anokye-labs/kbexplorer-template/issues/401) | Existing |
| Bespoke Copilot representation | [`kbexplorer-template#407`](https://github.com/anokye-labs/kbexplorer-template/issues/407) | Existing |

## Recommended sequencing

1. **Close the silent adoption gaps first.** Configurable structured-content
   paths, remote structured-content ingestion, typed edge preservation, and
   diagram rendering are the defects that make users distrust the system.
2. **Add diagnostics around every known footgun.** `kbx doctor` should explain
   source layout, mode parity, protocol/capability compatibility, and optional
   feature readiness before users debug the UI.
3. **Stabilize extension and compatibility contracts.** Provider factories,
   protocol versions, and capability flags should make custom repository
   integration supportable across independent CLI/template releases.
4. **Invest in representation-specific UX.** Host-themeable and Copilot-specific
   surfaces should reuse the same graph and viewers, but optimize interaction
   for narrow, agent-driven contexts.

## Definition of a paved path

A repository is on the paved path when:

- `kbx init` or equivalent setup records the selected template/version and
  validates compatibility.
- Structured descriptor input is configured, documented, and checked by
  `doctor`.
- Local and remote modes either render the same conceptual graph or clearly
  explain why a capability is unavailable.
- Authored markdown renders supported content blocks, including diagrams.
- The graph preserves typed and directional relationships emitted by providers.
- Extension points are documented through contracts and runnable examples.
- Users can choose the right representation target for their context without
  forking the graph model.
