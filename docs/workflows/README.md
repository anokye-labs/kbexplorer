# Workflows

Operational workflow documentation for running a kbx knowledge base day to day.
These are the concrete, repeatable processes — the "how" companion to the
persona-based [journeys](../journeys.md) and
[user stories](../user-stories.md) ("why").

| Document | What it covers |
|----------|---------------|
| [Creating a dataset](creating-a-dataset.md) | Standing up a new kbx knowledge base: `kbx init`, template strategy, content mode, first `derive`, first `build`. |
| [Hosting a dataset](hosting-a-dataset.md) | Deploying the built KB: GitHub Pages showcase, Azure App Service example, the search service. |
| [Updating a dataset](updating-a-dataset.md) | The refresh loop: `kbx derive`, the drift gate, incremental vs. full refresh, CI-driven cron/webhook refresh. |
| [Human approval workflow](human-approval-workflow.md) | How changes get human sign-off: `create_pr`, the consent gate, `refs #N` conventions, the conflation carve-out. |
| [Governance with rulesets](governance-with-rulesets.md) | CI gates, auto-merge automation, the conflation exception, `refs`-not-`closes` discipline. |
| [Search corpus updates](search-corpus-updates.md) | How `kbexplorer-search` derives SearchUnits, embedding vs. lexical providers, access-label exclusion, drift gate, serving. |
| [Human-in-the-loop ingestion](human-in-the-loop-ingestion.md) | MCP-based ingestion with human credentials: consent over elicitation, provider lineage, how it differs from autonomous discovery. |

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
