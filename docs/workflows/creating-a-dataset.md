# Creating a dataset

Standing up a new kbx knowledge base from scratch: initializing the project,
choosing a template strategy, configuring content and visual identity, deriving
the first entities, and building the explorer.

For _who_ uses kbx and _why_, see
[personas](../personas.md) and [user-stories](../user-stories.md) (story family
A — "First-time setup"). This doc covers the _how_.

## Prerequisites

- **Node.js >= 22** and **git** on the PATH.
- A GitHub repository you want to turn into a knowledge base.
- For fuzzy (LLM) phases (`generate`, `derive`): the
  [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/copilot-cli) on
  your PATH (`copilot --version`), or set `KBX_COPILOT_BIN`. Deterministic
  commands do not need it.

## 1. Initialize

From the root of your target repo:

```bash
npx @anokye-labs/kbx init
```

The interactive wizard auto-detects your git remote and branch and pre-fills
every prompt (owner, repo, branch, title, content mode, visual mode, theme).
Press **Enter** through all defaults to accept them, or override as needed.

### Non-interactive setup (CI / scripted)

Pass `--yes` to skip prompts entirely:

```bash
npx kbx init --yes
npx kbx init --yes --owner acme --repo widgets --title "Acme KB"
```

Without `--yes` on a non-TTY stdin, `init` exits with a reminder rather than
hanging.

Common flags (`npx kbx init --help` for the full list):

| Flag | Purpose |
|------|---------|
| `--owner`, `--repo` | GitHub owner/repo (auto-detected from git remote) |
| `--kb-branch` | Branch to read content from |
| `--title` | Knowledge base display title |
| `--content-mode <repo\|authored\|both>` | What content to ingest |
| `--content`, `--visual`, `--theme` | Content directory, visual mode, theme |
| `--runtime <copilot\|claude\|custom\|skip>` | LLM runtime for fuzzy phases |
| `--config <file>` | Load flag values from a JSON file |

### Template strategy

`init` installs the [kbexplorer-template](https://github.com/anokye-labs/kbexplorer-template)
SPA into `.kbx/`. Two install modes are available (ties to
[user-stories.md A2](../user-stories.md) — template strategy choice):

| Mode | Flag | What you get |
|------|------|--------------|
| **Submodule** (default) | _(none)_ | `.kbx/` is a pinned git submodule. `kbx update` bumps the pin. Best for tracking upstream. |
| **Vendor** | `--vendor` / `--no-submodule` | `.kbx/` is a plain copy (`.git` stripped). Best for copy-and-customize. |

Pin to a specific tag or branch:

```bash
npx kbx init --ref v1.2.0
npx kbx init --vendor --ref main
```

Both modes record the template origin in `.kbx.json` at your repo root:

```json
{ "template": "<url>", "ref": "v1.2.0", "refType": "tag",
  "resolvedCommit": "...", "mode": "submodule" }
```

Use a custom or org-internal template with `--template`:

```bash
npx kbx init --template https://github.com/my-org/my-template.git
```

## 2. Choose content mode

The `--content-mode` flag (or the interactive prompt) selects what content
populates the knowledge graph:

| Mode | Sources | When to use |
|------|---------|-------------|
| `repo` | GitHub Issues, PRs, commits, releases, file tree, README | Exploring an existing codebase |
| `authored` | Markdown files with YAML frontmatter from `content/` | Hand-authored knowledge bases |
| `both` | All of the above | Comprehensive view |

In `authored` mode, each `.md` file in the content directory becomes a graph
node. Frontmatter fields (`title`, `cluster`, `parent`, `connections`,
`entityType`, etc.) drive the graph structure. Starters for the five
organizational-layer descriptor kinds (person, squad, workstream, mission,
priority) are in
[`kbexplorer-cli/docs/templates/`](https://github.com/anokye-labs/kbexplorer-cli/tree/main/docs/templates).

## 3. Configure visual identity

The wizard offers a visual-mode and theme choice. Three built-in base modes
ship in code — `dark`, `light`, `sepia` — and named theme variants can be
declared in `content/config.yaml` under `theme.themes` or loaded from a
separate YAML file via `theme.themesFile`.

## 4. First content generation

If you start from an existing repo, `generate` bootstraps content:

```bash
npx kbx generate          # drives copilot -p -> catalogue.json -> content/ -> manifest
npx kbx generate --dry-run # preview the exact copilot command first
```

For hand-authored content, scaffold individual pages:

```bash
npx kbx scaffold <slug> --cluster <id> --title "My Page"
```

## 5. First derivation

If you have unstructured sources (`.docx`, prose `.md`, `.txt`), `derive`
extracts entities and relationships into committed JSON-LD artifacts:

```bash
npx kbx derive docs/org-chart.docx notes/teams.md
npx kbx derive docs/org-chart.docx --dry-run   # preview first
```

Each emitted node carries the F1 contract fields: a `kg://` identity URN
(`@id`), an open `@type`, a `@context`, and relationships mapped onto the
[six-relation taxonomy](https://github.com/anokye-labs/kbexplorer-core/blob/main/src/relations.ts)
(`leads | staffs | reports-to | structural | derived | deprecated`). The
committed artifact embeds a `source.ref` back to the originating document.

Derivation is idempotent: re-running on an unchanged source reuses the
embedded extraction intermediate (keyed by the source's SHA-256) and re-emits
**byte-identical** output without calling the LLM. This is the foundation of
the deterministic drift gate described in
[updating a dataset](updating-a-dataset.md).

## 6. First build

```bash
npx kbx build    # production build -> dist/kb/
```

Or start the dev server for iterating:

```bash
npx kbx dev      # regenerates manifest, starts Vite at :5173
```

## 7. Validate

```bash
npx kbx audit    # CI-grade structural lint (duplicate ids, broken parents, cycles)
npx kbx links    # soft graph-health report (orphans, weak clusters, coverage gaps)
npx kbx doctor   # diagnose runtime, MCP, template, adoption readiness
```

`audit` exits non-zero on errors and is suitable as a CI gate. `doctor`
diagnoses the full local setup across Runtime, MCP, Template, Adoption
readiness, Plugin, Sources, and Environment sections.

## What `init` creates

| Artifact | Purpose |
|----------|---------|
| `.kbx/` | The explorer template (submodule or vendor copy) |
| `.kbx.json` | Template origin, mode, and runtime config |
| `.env.kbx` | Gitignored environment variables |
| `.github/agents/` | kb-architect, kb-writer, kb-researcher agents |
| `.github/skills/kbx/` | kbx skill with focused references |
| `npm scripts` | Added to your `package.json` |

## Next steps

- [Hosting a dataset](hosting-a-dataset.md) — deploy the built KB.
- [Updating a dataset](updating-a-dataset.md) — the refresh loop.
- [Search corpus updates](search-corpus-updates.md) — build and serve
  semantic search over the graph.

---

> The architecture behind all of this is the four-layer model documented in
> [`docs/architecture.md`](../architecture.md): Sources -> Providers -> Engine
> -> Representation. The "index, don't migrate" principle
> ([kbexplorer#12](https://github.com/anokye-labs/kbexplorer/issues/12)) is
> why every workflow here adds to the graph without moving data out of its
> system of record.
