# Workflows

Operational workflow documentation for running a kbx knowledge base day to day.
These are the concrete, repeatable processes — the "how" companion to the
persona-based [journeys](../journeys.md) and
[user stories](../user-stories.md) ("why").

| Document | What it covers |
|----------|---------------|
| [Creating a dataset](creating-a-dataset.md) | Standing up a new kbx knowledge base: `kbx init`, template strategy, content mode, first `derive`, first `build`. |
| [Hosting a dataset](hosting-a-dataset.md) | Deploying the built KB: GitHub Pages showcase, any static host, the search service. |
| [Updating a dataset](updating-a-dataset.md) | The refresh loop: `kbx derive`, the drift gate, incremental vs. full refresh, CI-driven cron/webhook refresh. |
| [Human approval workflow](approving-a-change.md) | How changes get human sign-off: `create_pr`, the consent gate, `refs #N` conventions, the conflation carve-out. |
| [Governance with rulesets](rulesets-and-automation.md) | Governing a dataset repo: required checks, auto-merge lanes for agent PRs, the conflation exception, `refs`-not-`closes` discipline. |
| [Search corpus updates](updating-the-search-corpus.md) | How `kbexplorer-search` derives SearchUnits, embedding vs. lexical providers, access-label exclusion, drift gate, serving. |
| [Human-in-the-loop ingestion](ingesting-with-mcp.md) | MCP-based ingestion with human credentials: consent over elicitation, provider lineage, how it differs from autonomous discovery. |

## The paved path is an agent

The intended way to run these workflows is to **work with an agent** — the
kb-architect / kb-writer / kb-researcher agents that `kbx init` installs, or
any MCP client connected to `kbx mcp`. The agent drives the same affordances
documented here, behind the consent gate described in
[human approval workflow](approving-a-change.md). The `kbx` commands on
these pages document what runs under the hood, and remain the direct path for
scripting and CI.

**CLI spelling:** examples use plain `kbx`, assuming the CLI is installed
(globally via `npm install -g @anokye-labs/kbx`, or as a dev dependency).
`npx @anokye-labs/kbx` works for one-off runs on a repo with nothing
installed — bootstrap-time `init` is the only place you should need it.

## How to read the diagrams

Each page opens with a hand-authored, animated SVG in the visual language of
the overlay-network diagram
([#97](https://github.com/anokye-labs/kbexplorer/pull/97)): three horizontal
bands, with the numbered steps flowing left → right.

- **People · surfaces** (teal, top) — humans and the surfaces they act
  through: agent, canvas, review UI, terminal.
- **The overlay · kbx** (violet, middle) — what kbx itself does: genesis,
  derivation, staging, validation, projection.
- **Host · systems of record** (slate, bottom) — git, GitHub (PRs, Actions,
  rulesets, Pages), and external systems like Teams or SharePoint.

Steps pop in and edges draw in sequence, then the diagram holds, fades, and
rebuilds — the same build-up idiom as the overlay-network diagram, for the
same reason: a dataset *evolves*, it isn't a snapshot. Animation is pure CSS
and honors `prefers-reduced-motion`.

## Reading order

Start with [creating a dataset](creating-a-dataset.md) for initial setup, then
[hosting](hosting-a-dataset.md) for deployment. The day-to-day loop is
[updating](updating-a-dataset.md). The remaining docs cover specific concerns
in depth.

## Related documentation

- [Architecture](../architecture.md) — the four-layer model (Sources ->
  Providers -> Engine -> Representation).
- [Personas](../personas.md) · [Journeys](../journeys.md) ·
  [User stories](../user-stories.md) — who uses kbx and why.
- [Adoption paved path](../adoption-paved-path.md) — making kbx integration
  a guided path.
- [Surfaces](../surfaces.md) — the SPA showcase vs. the embeddable Copilot
  canvas.
