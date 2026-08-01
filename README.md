# Build a Guide — Auto-Generating Flowchart System

This is a complete, production-ready solution for **automatically generating interactive architecture flowcharts from source code**. It's designed to solve one of software engineering's most persistent problems: **documentation that goes stale the moment it's written**.

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

The complete prompt is embedded at the bottom of this page and includes:

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
