# CLAUDE.md – TECHNIK-Kalender

This file documents the repository structure, conventions, and development guidelines for AI assistants working on this codebase.

---

## Project Overview

**TECHNIK-Kalender** is a self-contained, single-file interactive Gantt chart application for planning multi-day school or event productions. It is written in vanilla HTML/CSS/JavaScript with no build toolchain.

**Primary file:** `Gannt-Diagramm.html` (the entire application – HTML, CSS, and JS in one file)

**Language:** German throughout (UI labels, variable comments, demo data)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML5 + CSS3 + ES2020+ JavaScript |
| Timeline library | [vis-timeline](https://visjs.github.io/vis-timeline/) v7+ via CDN (unpkg) |
| Data persistence | Browser `localStorage` + JSON export/import |
| Build toolchain | None – open the HTML file directly in a browser |
| Package manager | None |
| Testing | None |
| Backend | None – fully client-side |

---

## Repository Structure

```
TECHNIK-Kalender/
├── Gannt-Diagramm.html   # Complete application (HTML + CSS + JS)
└── README.md             # Minimal title only
```

---

## Architecture: Single-File SPA

The entire application lives in `Gannt-Diagramm.html`. The file is organised as:

1. **`<head>`** – CDN imports for vis-timeline JS + CSS, and all custom `<style>` rules
2. **`<body>`** – sticky `<header>` toolbar, `#timeline` container div, `<dialog>` modal editor
3. **`<script>`** – all JavaScript logic (storage, helpers, timeline init, event handlers, modal)

### Key JavaScript Sections (by source line)

| Lines | Section |
|---|---|
| 259–263 | Storage key constants |
| 265–291 | Utility helpers (`pad`, `uid`, `toLocalInput`, `parseLocalInput`, `dt`, `download`) |
| 294–362 | Color/class system (`GROUP_TO_WK`, `STATUS_CLASSES`, `buildClassName`, `applyCustomColor`) |
| 367–393 | Default demo data (`defaultGroups`, `defaultItems`) |
| 397–439 | Storage layer (`loadModel`, `saveModel`, `bootModel`) |
| 443–454 | Focus (viewport) persistence (`saveFocus`, `loadFocus`) |
| 459–524 | vis-timeline initialisation and focus helpers |
| 528–624 | Toolbar button event listeners |
| 629–728 | Modal editor (`openEditor`, form wiring, `onClose` callback) |

---

## Data Model

### localStorage Keys

| Key | Purpose |
|---|---|
| `school_gantt_v2` | Full model: groups, items, version, savedAt timestamp |
| `school_gantt_focus_v1` | Persisted timeline viewport (start/end ISO strings) |

### Model JSON Shape

```json
{
  "version": 2,
  "savedAt": "<ISO-8601>",
  "groups": [
    { "id": "string", "content": "string", "className": "wk-<id>" }
  ],
  "items": [
    {
      "id": "string",
      "group": "<group-id>",
      "content": "string",
      "start": "YYYY-MM-DD HH:MM",
      "end": "YYYY-MM-DD HH:MM",
      "type": "range",
      "className": "status-<x> wk-<x> [fmt-bold] [fmt-italic] [fmt-underline]",
      "style": "CSS string (optional, for custom colors)",
      "title": "tooltip text (optional)"
    }
  ]
}
```

### Groups (Gewerke)

| id | Display name | CSS class |
|---|---|---|
| `orga` | Orga / Ablauf | `wk-orga` |
| `buehne` | Bühne / Aufbau | `wk-buehne` |
| `licht` | Licht | `wk-licht` |
| `ton` | Ton | `wk-ton` |
| `video` | Video / Stream | `wk-video` |
| `catering` | Catering | `wk-catering` |
| `front` | FOH / Einlass | `wk-front` |

---

## CSS Class System

Item appearance is controlled entirely by CSS classes on each `vis-item`. Classes are built by `buildClassName()` and stored on each item.

### Status classes (background + border color)

| Class | Meaning | Color |
|---|---|---|
| `status-todo` | Not started | Gray |
| `status-doing` | In progress | Blue |
| `status-wait` | Blocked/waiting | Orange |
| `status-critical` | Critical path | Red |
| `status-done` | Completed | Green (dimmed) |

### Format classes (text styling)

| Class | Effect |
|---|---|
| `fmt-bold` | `font-weight: 800` |
| `fmt-italic` | `font-style: italic` |
| `fmt-underline` | `text-decoration: underline` |

### Gewerk classes (left-edge color bar via `box-shadow: inset 5px 0 0`)

`wk-orga`, `wk-buehne`, `wk-licht`, `wk-ton`, `wk-video`, `wk-catering`, `wk-front`

### Custom color (inline `style` attribute)

Applied by `applyCustomColor(styleStr, hex, mode)` where `mode` is one of: `none` | `bg` | `border` | `both`. This writes `background:` and/or `border-color:` inline CSS directly onto the item's `style` string, stripping any previously injected values first.

---

## Key Functions Reference

### Helpers

```js
uid()                        // Generate unique hex ID
pad(n)                       // Zero-pad to 2 digits
dt(y, m, d, h=0, min=0)      // Construct a local Date
toLocalInput(dt)             // Date → "YYYY-MM-DD HH:MM" string
parseLocalInput(s)           // "YYYY-MM-DD HH:MM" string → Date (null on error)
download(filename, text)     // Trigger browser file download
```

### Class management

```js
splitClasses(className)      // String → trimmed array
joinClasses(arr)             // Array → deduplicated space-joined string
removeAny(classes, blacklist)// Filter array, removing blacklisted entries
buildClassName({groupId, status, bold, italic, underline, existingClassName})
applyCustomColor(styleObj, hex, mode)
```

### Storage

```js
loadModel()   // Returns parsed model from localStorage or null
saveModel(groups, items)  // Serialises vis.DataSets back to localStorage
bootModel()   // Loads or seeds from defaults; returns { groups, items } DataSets
```

### Modal editor

```js
openEditor({ mode: "add"|"edit", item }, doneCb)
// Opens the <dialog>, populates form fields, calls doneCb(result|null) on close
```

---

## Development Workflow

### Running the app

No build step needed. Open `Gannt-Diagramm.html` directly in a modern browser:

```bash
# Any of the following work:
open Gannt-Diagramm.html          # macOS
xdg-open Gannt-Diagramm.html      # Linux
# Or serve via any static file server:
python3 -m http.server 8000
```

### Editing

All changes go into `Gannt-Diagramm.html`. There is no separate JS/CSS source — the file IS the source. Maintain the existing section order (style → body → script).

### No linting / CI / tests

There are no automated checks. Validate changes manually in the browser.

---

## Conventions & Coding Style

- **Vanilla JS only** – do not introduce npm, bundlers, or frameworks.
- **German UI strings** – all user-visible text stays in German.
- **Inline `<style>` and `<script>`** – keep everything in the single HTML file.
- **localStorage auto-save** – every mutation to `items` or `groups` DataSets triggers `saveModel` via the `items.on("*", ...)` / `groups.on("*", ...)` listeners. Manual saves in callbacks are belt-and-suspenders redundancy.
- **`buildClassName` is the single source of truth** for class strings on items. Never manually construct class strings; always go through this function.
- **Date handling** – always use `parseLocalInput` and `toLocalInput` for round-tripping dates through the modal form. Use `dt()` for hardcoded dates in source.
- **IDs** – generated with `uid()` (random hex + timestamp hex). Never use sequential integers for new items.
- **Modal pattern** – `openEditor` uses a callback (`done(result|null)`) instead of Promises. Follow this pattern if extending.

---

## Deployment

- Drop `Gannt-Diagramm.html` onto any static file host (GitHub Pages, Netlify, S3, local web server).
- No environment variables, no backend, no database.
- Requires a modern browser with CSS Grid, CSS custom properties, `localStorage`, and `<dialog>` element support (Chrome 99+, Firefox 98+, Safari 15.4+).
- CDN dependency: `https://unpkg.com/vis-timeline@latest` – requires internet access on first load; after that JS/CSS may be cached.

---

## Extending the App

When adding features, follow these patterns:

1. **New group/Gewerk:** Add an entry to `defaultGroups`, add a mapping in `GROUP_TO_WK`, and add CSS rules for both `.vis-item.wk-<id>` (box-shadow) and `.vis-label.wk-<id>` (border-left).
2. **New status:** Add an `<option>` to `#f_status`, add the class name to `STATUS_CLASSES`, and add a CSS rule for `.vis-item.status-<x>`.
3. **New item fields:** Extend the modal form HTML, populate in `openEditor`, read back in `onClose`, and store in the item object (it will be serialised automatically).
4. **Changing demo data:** Edit `defaultItems` / `defaultGroups`. These only apply when localStorage is empty or after a "Reset" action.

---

## Git Workflow

- Default branch: `main`
- Feature/AI work branches: `claude/<description>` (e.g. `claude/add-claude-documentation-Xjpug`)
- No CI pipeline is configured.
- Commit messages should be short and descriptive in English.
