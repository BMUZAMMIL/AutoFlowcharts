# Auto-Generating Flowchart System

This is a complete, production-ready solution for **automatically generating interactive architecture flowcharts from source code**. It's designed to solve one of software engineering's most persistent problems: **documentation that goes stale the moment it's written**. Download the index.html and it contains all the details you'll need with the prompt for the agent to build the system for your projects. If you wish to build the system for another project, not your active project that agent is working on then simply add extra message before the prompt to build the system for the project "your project folder location". 

---

## What This System Does

| Problem | This Solution |
|---------|---------------|
| Hand-drawn architecture diagrams rot as code changes | Diagrams regenerate from the code itself — they're **always current** |
| Expensive LLM calls for every diagram update | Pure regex scanner — **zero API cost**, ~130ms warm regeneration |
| Diagrams show only one level of detail | **Depth control (1–5)** — overview → classes → methods → internals → component effects |
| New developers spend weeks spelunking | Click any module to drill in, search to filter, share any view via **deep links** |
| Complex build steps, backend requirements | Single static HTML page — works from `file://`, GitHub Pages, or any static host |

---

## The Three-Part Pipeline

```
Your Source Code → Scanner → Data File → Viewer → Interactive Flowchart
```

### 1. Scanner (`scan-and-generate.mjs`)
- **Zero dependencies** — uses only Node.js standard library
- **Pure regex extraction** — imports, classes, methods, descriptions
- **Incremental cache** — `.arch-scan-cache.json` fingerprints files by `mtime+size`
- Full re-scan: ~130ms on warm cache
- `--no-cache` flag forces full rescan

### 2. Data Contract (`js/arch-data.js`)
The single source of truth — one JSON file containing everything:

```javascript
window.ARCH_DATA = {
  systems: [
    { id: "engine", title: "Engine", important: true, 
      desc: "Converts fuel into rotation", 
      modules: ["engine/engine.ts", "engine/fuel-injection.ts"] }
  ],
  nodes: {
    "engine/engine.ts": {
      kind: "module", label: "engine.ts", desc: "Combustion core",
      imports: ["engine/fuel-injection.ts", "electrical/battery.ts"],
      classes: [
        { label: "Engine", kind: "class", methods: [
          { label: "start", steps: [
            { label: "prime fuel pump", effects: ["electrical/battery.ts"] }
          ]}
        ]}
      ]
    }
  }
};
```

### 3. Viewer (`explorer.html`)
- **Tag cloud** — one chip per system, font size proportional to module count
- **Depth button + sidebar controls** (− / +) with auto-adjust clamping
- **Search filter** — filters modules by name or description
- **Click to drill** — expand any module to see its internals
- **Breadcrumb navigation** — Home / System / Module
- **Deep links** — `#system=engine&node=engine/engine.ts&depth=5`
- **Zoom toggle** — grows SVG width for detailed inspection
- **Animations** — staggered node entrance, flowing edges, glow on important nodes

---

## The Depth Model

| Level | What It Shows | Example (Engine) |
|-------|---------------|------------------|
| **1** | System overview — modules + import edges | `engine.ts`, `fuel-injection.ts`, `ignition.ts` |
| **2** | + Classes inside each module | `Engine`, `CoolingSystem` |
| **3** | + Methods inside each class | `start()`, `throttle()`, `checkOil()`, `stop()` |
| **4** | + Method internals (steps) | `start()` → prime fuel pump, crank starter, fire ignition coils… |
| **5** | + Component-level effects of each step | prime fuel pump → `battery.ts`, `wiring.ts` (electrical) |

**Auto-adjust**: The depth control clamps to the deepest level that actually has content — a module with no classes clamps to 1, a class with no methods clamps to 2, etc.

---

## The Big Promise

### One Prompt Builds Everything

At the bottom of this page is a **complete prompt** you can copy into any capable AI agent (Claude, ChatGPT, Codex, Hermes, OpenHuman…). The agent:

1. **Examines your project** — languages, structure, feasibility
2. **Builds the system** — scanner, generator, viewer, and demo dataset
3. **Runs it against your real code** — verifies all acceptance criteria

### Works For Any Codebase

| Scenario | What Happens |
|----------|--------------|
| **Full codebase** (any language) | Dynamic regex scanner builds live flowcharts |
| **Small project** (~50 files or single module) | LLM-based generation with user-provided API key |
| **No source code at all** (docs-only) | Agent reports why, builds nothing |
| **Language that can't be regex-scanned** | LLM-based generation with regenerate button |

### Key Design Constraints

- **Zero LLM calls in the pipeline** — the scanner runs entirely offline
- **Zero runtime dependencies** — Node standard library only
- **Zero build step** — works from `file://`, no backend
- **Works offline** — except for Mermaid CDN (which can be vendored)

---

## Why Flowcharts Matter

> **"Code is the most detailed description of a system ever written — which is exactly why it hides the big picture."**

A flowchart from code gives you a **map of the city that actually got built**:

- **Hot modules** — modules many others import (highest-risk, highest-value)
- **Cycles** — circular dependencies that signal missing abstractions
- **God modules** — one module with dozens of classes or hundreds of methods
- **Dead code** — modules nothing imports (deletion candidates)
- **Layering violations** — infrastructure leaking into everything

### Use It For

| Activity | How It Helps |
|----------|--------------|
| **Before writing a feature** | Check what the change touches — design on purpose, not by accident |
| **After a refactor** | Regenerate and diff — unintended new edges are the first cheap signal |
| **Onboarding** | New developer reads the map in 5 minutes, finds the right file in 5 more |
| **Code review** | Review the diff against the diagram — does the change respect the architecture? |
| **Architecture decisions** | One system, five zoom levels — architect, implementer, and debugger all served |

---

## How To Use It

1. **Copy the prompt** at the bottom of this page
2. **Paste into any capable agent** (Claude, ChatGPT, Codex, Hermes…) inside your project folder
3. **Run the scanner** on your codebase:
   ```bash
   node scan-and-generate.mjs /path/to/your/src
   ```
4. **Open `explorer.html`** — your project is now a live, navigable architecture map
5. **Regenerate whenever the code changes** — incremental cache keeps it fast
6. **Publish** — drop the folder on GitHub Pages, or open it straight from disk

---

## The Prompt

The complete prompt is below and includes:

- **Phase 0** — Examine the project, judge feasibility
- **Phase 1** — Build the scanner, generator, viewer, and demo dataset
- **Phase 2** — Run against the real project and verify all acceptance criteria
- **11 acceptance criteria** to verify against
- **Troubleshooting guide** for common rendering issues
- **LLM-based fallback path** for small projects or languages that can't be regex-scanned

It's designed to be **self-contained** — the agent needs nothing else to build the complete working system.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Scanner | Node.js (standard library only) |
| Rendering | Mermaid 11 (MIT-licensed, CDN) |
| Viewer | Vanilla HTML/CSS/JS |
| Animations | Pure CSS (no libraries) |
| Deployment | Any static host (GitHub Pages, S3, Netlify, file://) |

---

## In One Sentence

**A flowchart generated from the code gives you a map of the city that actually got built — so you can optimize the right streets, and never accidentally route a highway through someone's living room.**

---

## The Prompt

You are an autonomous agent. Your job: build an auto-generating flowchart system for a target project — software that scans a source codebase and produces interactive, depth-controlled architecture flowcharts that never go stale.

Work in three phases, in order:

1. **EXAMINE** the project and judge feasibility.
2. **BUILD** the system — but only if the examination says it's possible.
3. **RUN** it against the real project and **VERIFY** the results.

The pipeline you build must be zero-LLM (no API calls, no tokens), zero-dependency (Node.js standard library only for the scanner/generator), and static (plain HTML + JS + the Mermaid CDN for the viewer — no build step, no backend, works from file://).

---

## Phase 0 — Examine the project first (feasibility assessment)

Before writing any code, investigate the project you've been pointed at (your current working directory, or the path you were given). Report your findings:

* **Languages**: what languages does the source use? Estimate codebase size (file counts per language).
* **Structure**: where does the source live (`src/`, `lib/`, top-level `.ts`/`.py` files…)? What are the natural root namespaces (top-level source folders)?
* **Ignore list**: `node_modules`, `.git`, `dist`, `build`, `.next`, `coverage`, lockfiles, binaries, vendored code.
* **Extraction rules** for the language(s) found — regex-level, e.g.:
* **TypeScript/JavaScript**: ES `import`/`export` statements; `class Name` / `class Name extends Base`; method names inside class bodies; JSDoc first line.
* **Python**: `from x import y` / `import x`; `class Name`; `def name`; docstring first line.
* **Go**: `import "x"`; `type Name struct/interface`; `func (r Receiver) Name`; comment first line.
* **Other languages**: derive equivalents.


* **Feasibility gate**:
* **HARD STOP** — do NOT build anything, just report: there is no source code at all (docs-only or binary-only repo). Explain what you found and why nothing can be mapped.
* **SMALL PROJECT** → build the LLM-based flowchart generation system (section 7) instead of the dynamic scanner. Inform the user explicitly — in the README and in a banner on the page — that only a static flowchart system can be built for their project and why (a small codebase, roughly under ~50 source files or a single-module structure, doesn't give the dynamic scanner enough structure to be worth it).
* **CANNOT REGEX** → if the language's structure can't be captured reasonably with regex, do NOT hard-stop: build the LLM-based flowchart generation system (section 7) — the regenerate button exists exactly for this case.
* Otherwise proceed with the full dynamic system (sections 1–6).



---

## Phase 1 — Build the system

### 1. Data model (the contract)

Everything revolves around one JSON shape, emitted as `js/arch-data.js`:

```javascript
window.ARCH_DATA = {
  generatedAt: "ISO timestamp",
  systems: [ 
    { id: "engine", title: "Engine", important: true, desc: "...", modules: ["engine/engine.ts", "..."] } 
  ],
  nodes: {
    "engine/engine.ts": {
      id: "engine/engine.ts", 
      kind: "module", 
      label: "engine.ts", 
      path: "engine/engine.ts", 
      desc: "...",
      imports: ["engine/fuel-injection.ts"],
      classes: [ 
        { 
          label: "Engine", 
          kind: "class", 
          desc: "...", 
          methods: [
            { 
              label: "start", 
              kind: "method", 
              steps: [ 
                { label: "prime fuel pump", effects: ["electrical/battery.ts"] }, 
                "..." 
              ] 
            } 
          ] 
        } 
      ]
    }
  }
};

```

### 2. Scanner — `scan-and-generate.mjs` (zero-dependency Node script)

* Walks the target project root (accept the path as an argument; default to the script's folder).
* Roots: treat each top-level source folder as a root system namespace; files directly at the root belong to a `<root>` system.
* Ignore: the Phase-0 ignore list plus any binary/non-code files.
* Extraction: use the Phase-0 rules for the project's language(s) — regex only, no LLM, no AST parser. Resolve imports to filesystem-relative paths (strip extensions, resolve `./` and `../`, map to the module path if the file exists).
* Every scanned file becomes a module node; files with zero imports and zero classes are still listed (leaf modules).

### 3. Incremental cache

* Per-file fingerprints `{ mtimeMs, size }` in `.arch-scan-cache.json` next to the script.
* On re-run, only re-read files whose fingerprint changed; reuse the rest.
* `--no-cache` flag forces a full rescan.
* Target: warm regeneration (unchanged tree) under 200 ms.

### 4. Generator

* Emits `js/arch-data.js` in the viewer folder, using the exact `window.ARCH_DATA` shape above.
* Emit per-system `title` and `important` fields from the same script (data-driven presentation — the UI never hardcodes names).
* Print a summary line: `N systems / M files — warm|cold (X ms)`.

### 5. Viewer — `explorer.html` (single self-contained page)

Load `js/arch-data.js` + Mermaid 11 (one global `mermaid.initialize`; never per-chart `%%{init}`). Implement:

* **Tag cloud**: one chip per system; font size proportional to module count (log scale, ~12–20 px); color by root namespace; star prefix for important systems; chip text = `title || id`; tooltip = `Title — path · description`. Clicking a chip opens that system.
* **Depth control** (1–5: a depth button AND `−` / `+` buttons in a sidebar) with auto-adjust:
* Level 1 = system overview (modules + import edges)
* Level 2 = + classes
* Level 3 = + methods
* Level 4 = + method internals (the steps a method performs)
* Level 5 = + component-level effects of those steps.
* Auto-adjust: if the current view has no content at the selected depth, clamp to the deepest level that does have content (button, sidebar, label and status line all reflect the clamped value). Opening/selecting a system always starts at level 1.


* **Search filter**: text box filtering visible modules by name/description.
* **Click-to-drill**: clicking a module re-renders with that module focused; breadcrumb (`Home / system / module`) navigates back.
* **Deep links**: parse `#system=<id>&node=<relPath>&depth=<1-5>` (URL-decoded) on load and on hashchange; the boot sequence must call the renderer exactly once (a deep-linked page must never render twice at startup — double renders race Mermaid's concurrent-render guard and leave a blank diagram until the user interacts).
* **Overview cap**: systems with more than 60 modules render the first 60 and append `— showing first 60 modules` to the status line.
* **Status line**: `Depth N · X modules in system · click a node to drill in`.
* **Zoom button**: a two-action toggle — its text/icon always shows the action the NEXT click will perform, so the user always knows both states: `"🔍 Zoom"` (off → click zooms in) ↔ `"↩ Reset zoom"` (zoomed → click resets to 100%). The label must switch on every click. Zoom by growing the SVG width (e.g. 160%, with `max-width:none`, so the scrollable container can actually scroll the zoomed chart); never by transform-scaling the container (a CSS transform doesn't extend the scrollable overflow area, so an `overflow:auto` container just clips it).
* **Interactivity glue**: after injecting the rendered SVG, call `bindFunctions` on the container so Mermaid click handlers work.
* **Render id**: never pass the container element's id as the `mermaid.render` id — if an element with that id already exists, Mermaid removes it from the DOM and the diagram vanishes. Use a unique id (e.g. `"arch-chart"`) and inject `r.svg` into your own container.
* **Animations** (pure CSS, no libraries): nodes fade in staggered (per-node `animation-delay` after render; animate opacity ONLY — never transform, because CSS transform overrides the SVG `transform="translate(x,y)"` attribute and every node jumps out of its subgraph), arrows/edges flow continuously (infinite linear `stroke-dashoffset` animation applied purely via CSS on the `.edgePaths path.flowchart-link` selector — that is how mermaid 11 names every edge path, so a single rule animates all arrows; never try to add classes to edge elements after render and never target old mermaid class names, because the selector silently matches nothing and the arrows stay frozen; and never a one-shot draw-on, which freezes `dasharray` and breaks long edges), a slow glow pulse on important nodes, hover highlight; disable all animations under `prefers-reduced-motion`.
* **Tag cloud**: the chips are links added from the data, one per system, based on the important parts of the app — systems flagged `important:true` get a `★` and emphasis; every chip (label, title, description) is derived from the data file, never hardcoded.
* **Colors** — never the plain default Mermaid theme: dark palette; per-category `classDefs` (module/class/method/step/effect); a gold `classDef` for important systems; `linkStyle default` to color the edges; per-root accent colors in the tag cloud.
* Must work from `file://` (data file loaded via plain `<script src>`), no build step, no backend.

### 6. Bundled example dataset (the car demo)

Include a hand-authored sample dataset using car parts as the example domain — do NOT generate an actual demo codebase. The flowcharts and depth levels are examples only; this dataset exists so the viewer works out of the box and can be verified even before the real scan:

* Root systems: `engine`, `transmission`, `drivetrain`, `brakes`, `suspension`, `electrical`, `infotainment`, `body` (~2–3 modules each).
* Give at least one module classes with methods (e.g., `Engine` → `start`, `throttle`, `checkOil`, `stop`) and at least one class with no methods, so the depth auto-adjust is observable.
* Extend one method to full depth 5: internals (steps) with component effects (e.g., `start()` → `prime fuel pump` → `electrical/battery.ts`, `electrical/wiring.ts`).
* Include clamp demos at every level: a leaf module with no classes (clamps to 1), a method with no steps (clamps to 3), a method with steps but no effects (clamps to 4).
* Realistic cross-system imports (`engine` → `electrical/battery`, `brakes` → `drivetrain/wheels`, etc.).

### 7. LLM-based generation system (small projects / no dynamic generation)

For small projects, or any project that can't support dynamic (regex-scanned) flowchart generation, build the same viewer in static mode — but generate the charts with an LLM instead of a scanner:

* **INFORM THE USER**: the page shows a clear banner — `"Static mode — flowcharts are generated on demand. This project is too small for the dynamic scanner (or doesn't support regex-based generation)."`, and the README says the same. Never build a static system silently.
* **Same viewer, same quality**: reuse section 5's viewer — Mermaid rendering, animated/colored charts, depth levels, custom palette — the chart data is produced by an LLM instead of a scanner.
* **Regenerate button**: add a `"🔄 Regenerate flowcharts"` button next to the banner. Clicking it asks for an API key — the user's own key, e.g., Google AI Studio (Gemini 3.1 Flash) — and uses that key to call the LLM and regenerate the flowcharts. The key lives only in the user's browser: never commit it, never send it anywhere except the provider the user chose.
* **Keep it simple**: every project is different, so don't over-engineer. The LLM needs to see the code and return the same `window.ARCH_DATA` JSON contract (section 1); everything else — what you send it, how you prompt it, how you store the result — is your judgment, adapted to the project. If a generation fails, show a clear error and keep the last good chart.
* The regenerated charts follow the same rules as the scanner's output — same JSON contract, same rendering, same depth behavior. The only difference is who produced the data.

---

## Phase 2 — Run against the real project and verify

1. Run the scanner on the real project: `node scan-and-generate.mjs <project path>`. Fix extraction issues you find (this is the point of the examination — the regex rules must match the real code).
2. Generate `js/arch-data.js` and open `explorer.html`. Confirm the real systems appear in the tag cloud and depth levels render.
3. Verify ALL acceptance criteria below against the real output (the car demo dataset is only the fallback for any criterion that needs a known-good shape).

---

## Constraints

* Zero runtime dependencies, zero build step. Zero LLM calls EXCEPT the optional LLM-based generation system (section 7), which uses only a user-provided API key.
* No fabricated data: module/class/method/import extraction must reflect the scanned files.
* The data file is the single source of truth — the viewer must not hardcode system names, titles, or star lists.
* Keep the whole system under ~1,500 lines of source.

---

## Acceptance criteria (verify all before finishing)

1. `node scan-and-generate.mjs <project path>` produces `js/arch-data.js` and prints a summary line with real system/file counts.
2. Re-running with no changes is under 200 ms and reuses the cache (log line confirms).
3. `explorer.html` opens from disk: tag cloud shows the real systems (and the bundled car demo when no project data exists yet), depth button works.
4. Level 1 shows modules + import edges; level 2 classes; level 3 methods; level 4 method internals; level 5 component effects.
5. A system/module with no deeper content clamps the depth control down; selecting a system resets depth to 1.
6. Clicking a module drills in; breadcrumb returns; search filters.
7. Opening `explorer.html#system=<a real system>&node=<a real module>&depth=5` in a fresh tab renders the drilled view immediately (no interaction needed).
8. Back/forward and manual hash edits re-render via `hashchange`.
9. Works with no console errors.
10. Diagrams render with animations (staggered node entrance, flowing edges/arrows) and a custom color palette — never the plain default Mermaid theme.
11. Small projects (or projects where dynamic generation isn't possible) get the LLM-based generation system: the agent explicitly informs the user that only a static flowchart system can be built, the page shows the static-mode banner, and the Regenerate button rebuilds the flowcharts from a user-provided API key (e.g., Google AI Studio / Gemini 3.1 Flash) — the key is never committed to the repo.

---

## Troubleshooting — if a diagram looks wrong

Open the browser console first (criterion 9 is zero console errors — an uncaught exception names the exact line). Then:

* **Nodes overlapping, escaping their subgraph boxes, or clipped at the diagram's edge** → you animated CSS transform on Mermaid's SVG `.node` elements; CSS transform overrides the SVG `transform="translate(x,y)"` attribute, so nodes render in the wrong place. Animate opacity only — never put transform in node keyframes.
* **Edges with a permanent gap or missing middle** → a one-shot "draw-on" animation froze `stroke-dasharray` (`fill-mode:both`). Use a continuously flowing dash (infinite `stroke-dashoffset` on `.edgePaths path.flowchart-link`); never freeze the `dasharray`.
* **Arrows completely static while the nodes animate** → the flow animation was applied to a class mermaid 11 never creates (e.g., the old edge-pattern-dotted name). Mermaid 11 renders every edge as `path.flowchart-link` inside `g.edgePaths` — style exactly that selector with the flowing dash; do not post-process the rendered SVG to add classes to edges.
* **Diagram vanishes right after render** → the `mermaid.render` id collides with an id that already exists in the DOM (Mermaid removes that element), or `render()` ran twice at boot (the concurrent-render guard drops the second render, leaving a blank until interaction). Use a unique id; call render exactly once at startup.
* **Diagram stuck on "rendering…" or blank** → `mermaid.render()` rejected. Log the rejection and show `e.message` in the diagram area — never fail silently.
* **Mermaid failed to load** → the CDN script is blocked or you're offline. Show a visible fallback message, or vendor the script locally.
* **Long labels / deep-link URLs pushing out of cards** → allow wrapping (`overflow-wrap:anywhere` on code and links).
* **Depth button seems stuck at 2** → auto-adjust by design at the system overview; click into a module to reach depths 3–5.
* **Zoom button appears to do nothing** → the zoom class isn't actually applied to the diagram element (a no-op toggle), the zoom uses transform-scale on an `overflow:auto` container (clips the scaled chart without adding scrollbars), or the button label never switches (it must alternate `"🔍 Zoom"` ↔ `"↩ Reset zoom"` on each click). Zoom by growing the SVG width instead.
* **Chart renders but with default colors** → the `classDef` names don't match the classes applied in the diagram source; check both sides.

---

## Deliverables

`scan-and-generate.mjs`, `js/arch-data.js` (generated), `explorer.html`, `.arch-scan-cache.json` (after first run), and a short `README.md` with run instructions. If the feasibility gate stopped you, deliver a written assessment instead of code.
