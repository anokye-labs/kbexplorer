# Workflows

This directory documents the **operational workflows** of a kbx dataset — the
repeatable processes people and agents run against one, end to end. Where the
[journeys](../journeys.md) are persona narratives and the
[user stories](../user-stories.md) are the demand map, these pages are the
**process documentation**: deliberately persona-free, written in terms of roles
(the author, the reviewer, the operator, the agent) so they can be referenced
from issues, PRs, and runbooks without carrying narrative baggage.

Throughout, **dataset** means the versioned unit a kbx overlay is built from:
the authored content, the manifest, the `.kbx.json` configuration, and the
committed search index — everything that lives in git and moves through pull
requests together.

## The workflows

| Workflow | In one line |
|---|---|
| [Creating a dataset](creating-a-dataset.md) | Empty repo → living, queryable KB via the genesis state machine |
| [Hosting a dataset](hosting-a-dataset.md) | One repo, many representations, served as a static site |
| [Updating a dataset](updating-a-dataset.md) | Change → affected subgraph → regenerate → PR → merge on green |
| [Approving a change](approving-a-change.md) | The human approval gate: propose, validate, review, merge |
| [Rulesets & automation](rulesets-and-automation.md) | GitHub rulesets and Actions as the dataset's management plane |
| [Updating the search corpus](updating-the-search-corpus.md) | The index is versioned with the graph and gated by CI |
| [Human-in-the-loop ingestion via MCP](ingesting-with-mcp.md) | Credentialed systems of record enter the graph only through a person |

## How to read the diagrams

Every page embeds a hand-authored, animated SVG in the visual language of the
overlay-network diagram
([#97](https://github.com/anokye-labs/kbexplorer/pull/97), in flight): three
horizontal bands, with data flowing left → right in step order.

- **People · surfaces** (teal, top) — humans and the surfaces they act
  through: CLI, plugin, canvas, review UI.
- **The overlay · kbx** (violet, middle) — what kbx itself does: genesis,
  generation, scoping, staging, validation, projection.
- **Host · systems of record** (slate, bottom) — git, GitHub (PRs, Actions,
  rulesets, Pages), and external systems like Teams or SharePoint.

Steps pop in and edges draw in sequence, then the diagram holds, fades, and
rebuilds — the same build-up idiom as the overlay-network diagram, and for the
same reason: a dataset is something that *evolves*, not a snapshot. Animation
is pure CSS and honors `prefers-reduced-motion`.

## Status honesty

These pages document the **paved path kbx is building toward**. Some steps are
shipped, some are landing, and each page's *Traceability* section links the
stories and issues that deliver its steps — where a step is still in flight,
the linked issue is the source of truth for its status. When you document
behavior beyond these workflows, link the canonical contract in
[`kbexplorer-core`](https://github.com/anokye-labs/kbexplorer-core) rather
than restating it.
