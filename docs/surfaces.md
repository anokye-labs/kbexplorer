# The two surfaces

kbx renders one pure `KBGraph` ([architecture](architecture.md)) onto **two
genuinely different surfaces**. They look superficially similar — both are a
web UI showing the same graph — and that similarity is exactly what causes
confusion. They are not two versions of the same product, one newer than the
other. They are two different **Representation targets**
([`spa`](https://github.com/anokye-labs/kbexplorer-template/blob/main/src/representation/targets/spa.tsx)
and
[`copilot`](https://github.com/anokye-labs/kbexplorer-template/blob/main/src/representation/targets/copilot.tsx)),
built for different consumers, shipped by different mechanisms, and updated on
different schedules. **One changing does not imply the other changed.**

| | The SPA showcase | The embeddable Copilot canvas |
|---|---|---|
| **Consumer** | A human, full-viewport, browsing | An agent (Copilot), narrow side-panel, conversation-anchored |
| **Entry point** | `index.html` | `canvas.html` |
| **Representation target** | `spa` | `copilot` |
| **Hosting** | Static site on GitHub Pages | Served by `kbexplorer-cli` over a `127.0.0.1` loopback HTTP server, one per canvas instance |
| **Who opens it** | Anyone with the URL | An agent, via `open_canvas` / `invoke_canvas_action` inside the GitHub Copilot app |
| **Default landing view** | The constellation (force-directed graph) | The **anchor-first** view — an anchor node + its weight-ranked neighbors; the constellation is an optional zoom-out |
| **Navigation model** | Human drags/clicks the force graph | Agent steers via actions; a `kg://` chip re-anchors |
| **Update cadence** | Rebuilt by `.github/workflows/showcase.yml` on push to `main`, on demand, or nightly | Rebuilt whenever the template is rebuilt locally / the CLI's pinned template release changes |

## Surface 1 — the SPA showcase

This is the human-facing, full-page single-page app: the constellation /
overview / reading-view / HUD experience at
**[anokye-labs.github.io/kbexplorer/](https://anokye-labs.github.io/kbexplorer/)**.

It is built and deployed by
[`.github/workflows/showcase.yml`](../.github/workflows/showcase.yml) in *this*
repo: the workflow checks out `kbexplorer-template` pinned to a release tag
(`TEMPLATE_REF`, currently `v0.4.0`), checks out *this* repo as the content
host, bakes a manifest in **local mode**
(`VITE_KB_LOCAL=true`, `VITE_KB_HOST_ROOT=<this checkout>`) so the deployed
bundle needs no runtime GitHub token or API calls, builds `index.html` with
`vite build`, and deploys the result to GitHub Pages. It runs on push to
`main`, on `workflow_dispatch`, and on a daily schedule (the schedule exists
partly to backstop bot merges, which use `GITHUB_TOKEN` and don't emit a `push`
event).

`TEMPLATE_REF` is a manually-bumped pin, not `main` — the template's own release
notes advise host repos to pin to a tag. Bumping it is an ordinary,
issue-first PR in this repo (see
[kbexplorer#95](https://github.com/anokye-labs/kbexplorer/pull/95) and
[kbexplorer#99](https://github.com/anokye-labs/kbexplorer/pull/99) for two real
examples of that bump landing).

## Surface 2 — the embeddable Copilot canvas

This is the agent-facing surface: a panel a Copilot agent opens inside the
GitHub Copilot app via `open_canvas`, backed by `canvas.html` — an
**additive**, headless entry point the template build emits alongside
`index.html` (`vite.config.ts`'s `rollupOptions.input` lists both). The full
`index.html`/`main.tsx` path is untouched by anything below.

### Who serves it

`kbexplorer-cli` — not GitHub Pages, not any static host. The CLI's extension
declares a canvas
([`src/extension/canvas.js`](https://github.com/anokye-labs/kbexplorer-cli/blob/main/src/extension/canvas.js)
`buildCanvasOptions()`), and when an agent opens it, the CLI starts (or
rehydrates) **one loopback HTTP server per canvas `instanceId`**
([`src/extension/canvas-server.js`](https://github.com/anokye-labs/kbexplorer-cli/blob/main/src/extension/canvas-server.js)),
bound to `127.0.0.1:0` (an OS-assigned ephemeral port — never a routable
interface). That server serves whichever build directory it resolves
(`KBX_CANVAS_BUILD_DIR` override, else `<template>/dist`, else `<host>/dist/kb`)
and injects a boot-config script before the bundle executes:

```js
window.__KBX_CANVAS__ = {
  local: true,
  visualMode: 'inherit-host',
  searchServiceUrl: '<origin>/search',
  anchorNodeId, // optional — the node to focus on open
};
```

This loopback boundary is documented and frozen in the CLI's
[`docs/canvas-loopback-contract.md`](https://github.com/anokye-labs/kbexplorer-cli/blob/main/docs/canvas-loopback-contract.md):
`GET /` (the entry + boot config), `GET /manifest` + `/manifest/slice` (graph
data), `POST /search`, `GET /events` (SSE — `graph-updated` / `anchor`), and
`POST /affordance/:name` (the do-seam adapter — see
[architecture.md](architecture.md#the-do-seam--affordances-as-a-protocol-neutral-action-layer)).
**The CLI never renders; the template never touches disk or spawns servers** —
the only thing crossing the line is HTTP on that per-instance loopback origin.

### What it renders

`canvas.html` boots a headless shell
([`src/canvas/EmbeddableApp.tsx`](https://github.com/anokye-labs/kbexplorer-template/blob/main/src/canvas/EmbeddableApp.tsx) —
`FluentProvider` + `HashRouter` only, no HUD/favicon/dock padding) that resolves
the `copilot` Representation target
([`src/representation/targets/copilot.tsx`](https://github.com/anokye-labs/kbexplorer-template/blob/main/src/representation/targets/copilot.tsx)).
That target's default landing is **not** the constellation — it's the
**anchor-first view**
([`AnchorFirstView.tsx`](https://github.com/anokye-labs/kbexplorer-template/blob/main/src/representation/targets/copilot-home/AnchorFirstView.tsx)):
the anchor node rendered through its normal node-type viewer, its
weight-ranked neighbors expanded inline up to `DEFAULT_MAX_EXPANDED` (6), and
every neighbor beyond that as a navigable `kg://` chip. Clicking a chip
re-anchors (`/node/:id`); the constellation is reachable as an explicit
zoom-out, not the landing.

### Theming

The canvas inherits the host's visual identity rather than shipping its own:
`useCanvasTheme` resolves Fluent design tokens from CSS variables the host
mirrors onto the iframe's `<html>` element, with a `MutationObserver` that
re-resolves on host theme switches, falling back to the template's own
configured theme when nothing is mirrored.

## Why the distinction matters

The two surfaces share almost everything **except** the one thing people
assume they share: what gets rendered right now. Both reuse the same pure
`KBGraph`, the same `kg://` identity scheme, the same six-member relation
taxonomy, and — as of the `copilot` target's initial cut — the same node-type
viewers as the `spa` target. But:

- **They deploy independently.** The SPA showcase redeploys when *this* repo's
  `TEMPLATE_REF` is bumped or its content changes. The canvas redeploys
  whenever whoever installed the CLI updates their pinned template build. A
  change to one is invisible on the other until someone explicitly re-pins.
- **They default to different views on purpose.** The SPA's constellation is
  built for a human with a mouse and a full viewport to *explore*. The
  canvas's anchor-first view is built for a conversation where the agent
  already knows what it's talking about and the panel should confirm, not
  make the human search for it.
- **The canvas is not "the SPA in an iframe."** It is a distinct
  `Representation` target (`copilot`, registered by
  [template#442](https://github.com/anokye-labs/kbexplorer-template/pull/442))
  that currently reuses the `spa` target's node viewers, but is free to diverge
  — and already has, via the anchor-first landing
  ([template#443](https://github.com/anokye-labs/kbexplorer-template/pull/443)).
- **The canvas epic is still open.** Only the anchor-first home view has
  shipped so far out of [template#407](https://github.com/anokye-labs/kbexplorer-template/issues/407)'s
  six sub-issues — agent actions (anchor/expand/trace/filter beyond the
  do-seam's HTTP contract), bidirectional click→chat, an affordance-aware
  node launchpad, and a fully conversation-shaped shell are designed but not
  yet built. See [history.md](history.md) for the verified, dated account.

## Run it yourself

To build and serve the canvas locally against your own template build, see
[canvas-dev-loop.md](canvas-dev-loop.md).
