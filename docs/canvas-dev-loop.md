# Run it yourself — the canvas dev loop

This is a **verified** workflow: every command below was actually run against
fresh clones of `kbexplorer-template` and `kbexplorer-cli` while writing this
document, and the resulting canvas panel was opened and its served HTML
inspected to confirm it was the real local build (not a fallback page). If a
future CLI/template change breaks a step, please update this doc rather than
leaving it stale.

Use this loop to iterate on the embeddable Copilot canvas
([surfaces.md](surfaces.md#surface-2--the-embeddable-copilot-canvas)) without
publishing a template release or reinstalling the CLI.

## Prerequisites

- Node.js (tested with v26.3.0) and npm.
- A local clone of [`kbexplorer-template`](https://github.com/anokye-labs/kbexplorer-template).
- A local clone of [`kbexplorer-cli`](https://github.com/anokye-labs/kbexplorer-cli)
  (only needed if you want to exercise the CLI's real loopback server rather
  than a toy server — see step 3).
- A repo to serve as KB host content. Any repo with a `content/` directory
  works; this walkthrough uses a checkout of this repo (`kbexplorer`) as the
  host, the same way `.github/workflows/showcase.yml` does.

## 1 — Install and build the template in local mode

```powershell
cd path\to\kbexplorer-template
npm ci
```

Generate a manifest in **local mode**, pointed at your host repo's content
(`VITE_KB_HOST_ROOT`) — this is the same pair of env vars
`.github/workflows/showcase.yml` sets for the GitHub Pages showcase build:

```powershell
$env:VITE_KB_LOCAL = 'true'
$env:VITE_KB_HOST_ROOT = 'path\to\kbexplorer'   # or any repo with content/
node scripts/generate-manifest.js
```

Then build:

```powershell
npx vite build
```

Confirm both entry points were emitted — `index.html` (the SPA showcase) and
`canvas.html` (the embeddable canvas) — in `dist/`:

```powershell
Get-ChildItem dist\*.html
# index.html
# canvas.html
```

If only `index.html` appears, your template checkout predates
[template#441](https://github.com/anokye-labs/kbexplorer-template/pull/441)
(the additive `canvas.html` entry) — pull a newer `main` or a `v0.4.0`+ tag.

## 2 — Install the CLI's dependencies

The canvas is served by `kbexplorer-cli`'s loopback server, so its
dependencies need to be installed too (frontmatter parsing, etc.), even though
you won't run `kbx` as a CLI in this workflow:

```powershell
cd path\to\kbexplorer-cli
npm ci
```

## 3 — Wire a session-scoped Copilot CLI extension

Extensions live at `.github/extensions/<name>/extension.mjs` (project-scoped)
or under the user/session extensions directory. For iteration, a
**session-scoped** extension is convenient — it's discarded when the session
ends and doesn't touch the repo. Scaffold one, then replace its body with a
thin wrapper around the CLI's real `buildCanvasOptions()` — reusing the exact
loopback server code the shipped extension uses, rather than hand-rolling a
toy HTTP server:

```js
// extension.mjs
import { joinSession, createCanvas } from "@github/copilot-sdk/extension";
import { pathToFileURL } from "node:url";

// Windows ESM requires a file:// URL for absolute paths — a bare drive-letter
// path ("C:/...") is rejected by the default loader.
const CLI_CANVAS_JS = pathToFileURL(
  "C:/path/to/kbexplorer-cli/src/extension/canvas.js",
).href;
const { buildCanvasOptions } = await import(CLI_CANVAS_JS);

// Tells canvas-server.js's defaultResolveBuildDir() which dist/ to serve
// (must contain canvas.html or index.html). Set before buildCanvasOptions()
// runs so the registry's resolution picks it up on open().
process.env.KBX_CANVAS_BUILD_DIR = "C:/path/to/kbexplorer-template/dist";

const session = await joinSession({
  canvases: [createCanvas(buildCanvasOptions())],
});
```

Reload extensions, then open the canvas (`kbexplorer` is the CLI's fixed
canvas id — see `KBX_CANVAS_ID` in `src/extension/canvas.js`):

```
open_canvas(canvasId: "kbexplorer", instanceId: "<any-handle>")
```

This should return a `url` like `http://127.0.0.1:<ephemeral-port>`. Verify the
real build is being served (not the built-in fallback placeholder) by
fetching it and checking for the injected boot config:

```powershell
Invoke-WebRequest -Uri "http://127.0.0.1:<port>" -UseBasicParsing |
  Select-Object -ExpandProperty Content |
  Select-String -Pattern "__KBX_CANVAS__"
```

You should see something like:

```html
<script>window.__KBX_CANVAS__={"local":true,"visualMode":"inherit-host","searchServiceUrl":"http://127.0.0.1:<port>/search"};</script>
```

If instead you see `data-kbx-fallback="true"`, `KBX_CANVAS_BUILD_DIR` isn't
pointing at a directory containing `canvas.html`/`index.html` — re-check the
path and that step 1's build actually succeeded.

## 4 — Iterate: rebuild, refresh

`canvas-server.js` resolves the build directory **once**, the first time a
given canvas `instanceId` is opened, but reads each file fresh
(`readFileSync`) on every request. That means:

- **Changing template source and rebuilding** (`npx vite build` again) is
  picked up on the **next page load** of an already-open panel — no need to
  close and reopen the canvas, just reload it.
- **Switching which entry file exists** (e.g. a build that stops emitting
  `canvas.html`) is *not* picked up without closing and reopening the canvas
  (a new `open()` call re-resolves the build dir).

A typical loop:

```powershell
# 1. edit kbexplorer-template source
# 2. rebuild
cd path\to\kbexplorer-template
npx vite build
# 3. reload the already-open canvas panel in the Copilot app
```

## 5 — Clean up

Session-scoped extensions are discarded automatically when the session ends.
To stop early, remove the extension's `.mjs` file and reload extensions; the
loopback server for any open canvas instance is torn down when the canvas is
closed (`onClose`).

## What this does *not* cover

This loop serves a **static** local build — it does not exercise:
- the do-seam's `/affordance/:name` route beyond what `buildCanvasOptions()`
  wires by default (no `actions` are declared in the snippet above — see
  [architecture.md](architecture.md#the-do-seam--affordances-as-a-protocol-neutral-action-layer)
  for the extension-tool adapter, which is a separate wiring concern from the
  canvas declaration itself);
- a real host repo's `dist/kb` build path (`<cwd>/dist/kb`, the third
  resolution fallback in `defaultResolveBuildDir`) — this walkthrough always
  used the explicit `KBX_CANVAS_BUILD_DIR` override;
- hot module reloading — this is a production `vite build`, not `vite dev`;
  each iteration is a full rebuild.
