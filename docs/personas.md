# The people who use kbx

kbx turns scattered organizational knowledge — issues, docs, work items, decisions,
people, services — into **one navigable, queryable graph**, stored in git and
rendered through pluggable surfaces. This page introduces the **cast**: the personas
referenced throughout the [journeys](journeys.md) and the [user-story demand
map](user-stories.md).

Personas are not roles in a permission system; they are *who shows up and what they
are trying to do*. They exist so that every capability we build can be checked against
a real human need — and so that claims like "works for non-GitHub sources" or "usable
without the CLI" have someone concrete to fail.

| Persona | In one line | Touches the CLI? | Drives |
|---|---|---|---|
| **Dana** | KB owner / author | Yes | Genesis, curation, generation |
| **Ravi** | Developer / contributor | Sometimes | Incidental nodes, refresh |
| **Mei** | Newcomer / learner | **Never** | Sense-making, search |
| **Lena** | Lead / reviewer | Rarely | Freshness, cross-cutting questions |
| **Copilot** | The agent | n/a (it *is* a client) | Grounded answers, actions, the do-seam |
| **Sam** | Plugin / platform operator | Yes | Install, distribute, wire credentials |
| **Noor** | Confluence analyst | **No git / Node / terminal** | Non-repo genesis, import |
| **Morgan** | Org modeler | Yes | Multi-source unification (the headline) |
| **Priya** | Azure DevOps / Jira operator | Yes | Work-item ingestion |
| **Kenji** | Perforce Helix Core dev | Yes (p4, not git host) | Host-decoupling |
| **Elena** | Compliance / governance | Rarely | Provenance, access, approvals |
| **Quinn** | Non-engineer knowledge worker | **Never** | One-address truth, propose-don't-overwrite |

---

## The core cast

### Dana — KB owner / author
Curates the structure of the knowledge base, runs `generate`, and owns the visual
identity and correctness of the result. Dana decides the template strategy and the
node-type model. When the KB looks wrong or goes stale, it is Dana's problem.
**Primary journeys:** [greenfield genesis](journeys.md#j1-greenfield-genesis).

### Ravi — developer / contributor
Lives in the repository and wants the KB to stay current with his pull requests. Ravi
contributes nodes *incidentally* — a code change spawns or updates a node — rather
than sitting down to author the graph. He cares that refresh is cheap and that the KB
never silently rots.

### Mei — newcomer / learner
**Never touches the CLI.** Mei makes sense of an unfamiliar system through search,
focused reading views, and the constellation — starting from an anchor, not a
hairball. Mei is the test for "usable without the terminal": if a capability requires
`npx`, Mei cannot reach it.

### Lena — lead / reviewer
Asks cross-cutting questions across clusters, wants freshness and health signals, and
relies on decision records. Lena reads the KB to make decisions; she rarely edits it.

### Copilot — the agent
A first-class actor, not a bystander. Copilot consumes the KB as grounded context,
drives the canvas through actions, calls the [MCP do-seam](roadmap.md) tools, and can
be handed work — but only the work a resource *advertises*. Copilot is the reason the
"do-seam" and the consent rules exist.

### Sam — plugin / platform operator
Installs and distributes the plugin across repos and scopes, wires the MCP
configuration and credentials (bring-your-own `gh` and provider keys), and runs the
surfaces. Sam is who the distribution and `doctor`/preflight stories serve.

---

## The non-GitHub fleet

These personas exist to keep kbx honest about its generality. Every "open kind / open
affordance / any source" claim is tested against them. If a journey for these people
cannot be written without GitHub-specific assumptions, the generality is aspirational.

### Noor — Confluence analyst
Imports from Confluence / Notion / Drive and wants queryable org context — but may
have **no git, no Node, and no terminal at all**. Noor breaks the CLI-first genesis at
step one and forces the *plugin-brokered* path (and a backing store that is not a
committed repo).

### Morgan — org modeler
Maintains teams, systems, owners, missions, and decisions across **multiple** sources.
Morgan is the headline use case: *many sources → one graph* by **connection, not
merge** — minting reference edges between distinct artifacts and conflating shared
people/teams/services.

### Priya — Azure DevOps / Jira operator
Maps work items, epics, iterations, risks, and ownership from Azure DevOps / Jira.
Priya proves that a non-GitHub system of record is just another source through a
generic provider.

### Kenji — Perforce Helix Core developer
Works in Perforce — no GitHub host. Changelists, streams, and assets; "affected" is a
changelist/shelf/label range, not a git diff. Kenji proves the host can be decoupled
while git stays the optional backing store.

### Elena — compliance / governance
Audits provenance, stale decisions, approvals, and access boundaries. A *unified* KB
is a new, broader access boundary — Elena is the reason access is labeled, generated
content needs a human approval gate, and the sampling bridge needs consent + cost
disclosure.

---

## The non-engineer

### Quinn — knowledge worker
Lives in the spreadsheet, builds the deck, iterates the doc. Quinn is not an engineer
and never will be. The Resource-Oriented framing in [#12's
discussion](https://github.com/anokye-labs/kbexplorer/issues/12) is, at heart, *for
Quinn*: make the structured part of what Quinn already writes **addressable** (one
thing, one address — stop reconciling the doc, the sheet, and the slide),
**navigable** (follow meaning, not folders), **time-aware** (the system remembers
*when*, not just *what*), and **safe to change** (propose, don't overwrite). The tool
meets Quinn where they are.

---

## Where personas show up

- Each persona drives one or more end-to-end [journeys](journeys.md).
- Each [user story](user-stories.md) names the persona it serves.
- The work that delivers each story is tracked under the demand-map umbrella
  [#23](https://github.com/anokye-labs/kbexplorer/issues/23).
