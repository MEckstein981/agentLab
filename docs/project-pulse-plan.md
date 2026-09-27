# Project Pulse — Implementation Plan

## 1. Summary

Project Pulse is a static, learner-facing dashboard that renders a portfolio of projects as cards with status and priority indicators. The frontend is plain HTML/CSS with a small vanilla-JS glue script that fetches a local `project-data.json` file and renders cards into a Designer-owned semantic template. Delivery is split cleanly between the Designer (markup + CSS + a documented render contract) and the Coder (JSON data, render script, and `.vscode/launch.json`) so both specialists can work in parallel without touching each other's files. The Orchestrator will run the result in a Codespace via the launch config so learners open directly to a working dashboard.

## 2. File assignments

| File | Owner | Purpose | Contracts / deterministic hooks |
|---|---|---|---|
| `app/index.html` | Designer | Semantic dashboard shell, card template, script tag that loads the render glue and JSON. | Root `<main class="dashboard">`, `<header class="dashboard__header">`, `<section class="dashboard__grid" id="projectGrid">` as the render target. `<template id="projectCardTemplate">` with `.project-card`, `.project-card__title`, `.project-card__owner`, `.project-card__status`, `.project-card__priority`, `.project-card__progress`, `.project-card__progress-bar`, `.project-card__updated`. Loads `./styles.css` and `./app.js` (deferred). Fetches `./project-data.json`. |
| `app/styles.css` | Designer | Visual system: layout, cards, status badges, priority treatment, responsive grid, typography, focus states. | Selectors: `.dashboard`, `.dashboard__header`, `.dashboard__grid`, `.project-card`, `.project-card--priority-high/medium/low`, `.project-card__status--active/at-risk/blocked/done`, `.project-card__progress`, `.project-card__progress-bar`. Responsive breakpoints: ≥1024px 3-col, 640–1023px 2-col, <640px 1-col. |
| `app/project-data.json` | Coder | Sample project records that drive the dashboard. | Strict JSON object `{ "projects": [...] }`. Fields: `id`, `name`, `status`, `priority`, `owner`, `progress`, `updated`. Enumerations: `status ∈ {active, at-risk, blocked, done}`, `priority ∈ {high, medium, low}`. |
| `.vscode/launch.json` | Coder | One-click preview in Codespaces so learners see the dashboard, not a directory listing. | Strict JSON, no comments. `cwd` = `${workspaceFolder}/app`. Opens `index.html`. Deterministic `name` (e.g. `"Preview Project Pulse"`), deterministic port if a server is used, and URL ending in `/index.html`. |

Also created by the Coder alongside `project-data.json` (implicit, non-overlapping with Designer files): `app/app.js` — small render glue script referenced by `index.html`. Called out explicitly here so ownership is unambiguous.

## 3. Designer responsibilities

**`app/index.html`**

- Semantic structure:
  - `<header class="dashboard__header">` with dashboard title ("Project Pulse"), short subtitle, and a live region `<p class="dashboard__meta" aria-live="polite">` for "Showing N projects" (populated by the render script).
  - `<main class="dashboard">` containing `<section class="dashboard__grid" id="projectGrid" aria-label="Projects">` — the render target.
  - `<template id="projectCardTemplate">` containing the exact card DOM the Coder's script clones and fills:
    - `<article class="project-card" data-priority="" data-status="">`
    - `<h2 class="project-card__title">`
    - `<p class="project-card__owner">`
    - `<span class="project-card__status">` (badge)
    - `<span class="project-card__priority">` (pill/flag)
    - `<div class="project-card__progress" role="progressbar" aria-valuemin="0" aria-valuemax="100">` containing `<div class="project-card__progress-bar">`
    - `<time class="project-card__updated">`
  - Empty-state element `<p class="dashboard__empty" hidden>No projects to show yet.</p>` toggled by the script.
- Loads `./styles.css` in `<head>` and `./app.js` with `defer` before `</body>`.
- Sets `<html lang="en">`, viewport meta, sensible `<title>`.

**`app/styles.css`**

- Deterministic hooks listed in §2, plus modifier classes driven by `data-status` and `data-priority` attributes (so Designer can style without Coder branching in JS).
- Visual system: rounded corners, subtle shadow, generous spacing, clear type scale, distinct color per status (accessible contrast ≥ 4.5:1 for text, ≥ 3:1 for badge backgrounds vs. surface), priority hierarchy via color + iconography/typography (not color alone).
- CSS Grid layout for `.dashboard__grid` with `minmax` for responsive card widths; explicit breakpoints at ~640px and ~1024px.
- Focus-visible outlines on interactive elements; respects `prefers-reduced-motion`.

**Consumption contract with Coder**

- Designer publishes and freezes these before Coder wires rendering:
  1. Template element `id="projectCardTemplate"` with exact class names above.
  2. Render target `id="projectGrid"`.
  3. Empty-state element `class="dashboard__empty"` toggled via `hidden` attribute.
  4. Card driven by `data-status` and `data-priority` attributes on the root `.project-card`.
  5. Meta live region `class="dashboard__meta"`.

## 4. Coder responsibilities

**`app/project-data.json`**

- Strict JSON, UTF-8, no trailing commas, no comments.
- Shape:

  ```json
  {
    "generatedAt": "2026-09-27T00:00:00Z",
    "projects": [
      {
        "id": "pp-001",
        "name": "Onboarding Revamp",
        "status": "active",
        "priority": "high",
        "owner": "Mona",
        "progress": 62,
        "updated": "2026-09-24"
      }
    ]
  }
  ```

- Enumerations must match Designer's CSS hooks exactly: `status ∈ {active, at-risk, blocked, done}`, `priority ∈ {high, medium, low}`.
- Include 6–8 realistic sample records covering every status and priority at least once, plus one long-name entry and one with `progress: 0` and one with `progress: 100` to exercise edge visuals.
- `updated` is ISO date (`YYYY-MM-DD`); script formats for display.
- `progress` is integer 0–100.

**`app/app.js` (render glue)**

- Vanilla JS, no dependencies.
- `fetch('./project-data.json')` → parse → validate minimally (array present, required fields per record) → for each project clone `#projectCardTemplate`, set text content and `data-status`/`data-priority`, set `aria-valuenow` on the progressbar, width on `.project-card__progress-bar`, and append to `#projectGrid`.
- Update `.dashboard__meta` with count; toggle `.dashboard__empty` `hidden` based on count.
- Handle fetch failure and JSON parse failure by rendering a visible, accessible error message inside `#projectGrid`.
- Skip malformed records (missing required fields) and log a single `console.warn` summarizing skipped entries.

**`.vscode/launch.json`**

- Strict JSON, no comments.
- One configuration:
  - `name`: `"Preview Project Pulse"`
  - Uses a Codespaces-compatible browser debug config to open `index.html`.
  - `cwd`: `"${workspaceFolder}/app"`.
  - `url` (or `file`) pointing at `index.html` served from `app/` so the browser opens the dashboard, not a directory listing.
  - If a static server is required for `fetch()` to work, use a deterministic port (e.g. `5173` or `8080`).

## 5. Ordered implementation steps

1. **Freeze the render contract** — Owner: Orchestrator (from this plan). Files touched: none. Inputs: agent specs, this plan. Outputs: agreed class names, template id, JSON schema, enums.
2. **Designer builds shell + template** — Owner: Designer. Files: `app/index.html`. Inputs: step 1. Outputs: semantic HTML with `#projectGrid`, `#projectCardTemplate`, `.dashboard__meta`, `.dashboard__empty`, and script/style tags referencing `./styles.css` and `./app.js`.
3. **Designer builds visual system** — Owner: Designer. Files: `app/styles.css`. Inputs: step 1. Outputs: responsive card grid, status/priority styling, badges, progress bar, focus states, empty state.
4. **Coder authors sample data** — Owner: Coder. Files: `app/project-data.json`. Inputs: step 1 schema/enums. Outputs: 6–8 valid sample records covering all enum values and edge visuals.
5. **Coder writes render glue** — Owner: Coder. Files: `app/app.js`. Inputs: steps 1, 2 (template/hook names), 4 (schema). Outputs: fetch + validate + render + empty/error handling + meta count.
6. **Coder adds launch config** — Owner: Coder. Files: `.vscode/launch.json`. Inputs: step 2 (existence of `app/index.html`). Outputs: deterministic one-click preview.
7. **Orchestrator integration verification** — Owner: Orchestrator. Files: none. Inputs: steps 2–6. Outputs: signed-off working dashboard per §8.

## 6. Dependencies

- Step 2 (HTML) depends on Step 1 (frozen hook names).
- Step 3 (CSS) depends on Step 1 (frozen hook names) — does **not** require Step 2's file to exist, only the contract.
- Step 4 (JSON) depends on Step 1 (schema/enums).
- Step 5 (`app.js`) depends on Step 1 (contract), Step 2 (template ids/classes present in `index.html`), and Step 4 (concrete JSON to test against).
- Step 6 (`launch.json`) depends on Step 2 (`app/index.html` must exist at the path the config opens).
- Step 7 depends on Steps 2–6.
- Cross-file runtime: `index.html` references `./styles.css`, `./app.js`, and `./project-data.json` — all must live in `app/`.

## 7. Parallel vs sequential work

**Can run in parallel after Step 1:**

- Step 2 (Designer, `app/index.html`) ‖ Step 3 (Designer, `app/styles.css`) ‖ Step 4 (Coder, `app/project-data.json`).
  - Justification: disjoint files, and the frozen contract from Step 1 removes coupling. CSS targets contract selectors, not step 2's concrete markup.

**Must run sequentially:**

- Step 5 (`app/app.js`) after Steps 2 and 4. Justification: script asserts against real template DOM and real JSON.
- Step 6 (`.vscode/launch.json`) after Step 2. Justification: launch target file must exist.
- Step 7 after everything. Justification: integration check.

**Optional parallelism:** Step 6 can start in parallel with Step 5 once Step 2 lands, because `.vscode/launch.json` and `app/app.js` have disjoint file scopes.

## 8. Validation expectations

Automated / structural:

- `app/project-data.json` parses via `JSON.parse` (or `python -m json.tool`) with zero errors.
- `.vscode/launch.json` parses as strict JSON (no comments), and its `cwd` is `${workspaceFolder}/app` and target is `index.html`.
- HTML is well-formed; all referenced files (`styles.css`, `app.js`, `project-data.json`) resolve.

Manual (learner runs in Codespace):

1. Open the "Preview Project Pulse" launch config → browser opens directly on the dashboard (not a directory listing).
2. Cards render: one per record in `project-data.json`; meta text reads "Showing N projects" with matching count.
3. Status badges are visually distinct across `active`, `at-risk`, `blocked`, `done`.
4. Priority treatment is visually distinct across `high`, `medium`, `low` and does not rely on color alone.
5. Progress bar widths match `progress` values; `aria-valuenow` matches.
6. Resize to ~1200px, ~800px, ~400px — grid reflows to 3 / 2 / 1 columns; no horizontal scroll; long project names wrap gracefully.
7. DevTools console shows no errors on load.
8. Temporarily replace `project-data.json` with `{"projects": []}` → empty state message shows.
9. Temporarily break JSON (e.g. trailing comma) → user-visible error message renders in `#projectGrid`.
10. Keyboard: Tab reaches any interactive elements with visible focus outline.

## 9. Edge cases and risks

- **Empty data**: `projects: []` → show `.dashboard__empty`, meta reads "Showing 0 projects".
- **Malformed JSON / fetch failure**: show accessible error message; do not leave grid blank without explanation.
- **`file://` fetch restriction**: opening `index.html` directly from disk may block `fetch('./project-data.json')`. Launch config should open via a local static server (deterministic port) rather than `file://`.
- **Long project names / owners**: CSS must handle wrapping, no overflow off the card.
- **Many projects (50+)**: script should build a `DocumentFragment` and append once.
- **Missing optional fields**: script tolerates missing `owner` or `updated` by hiding those nodes; missing required fields (`name`, `status`, `priority`) → skip record with `console.warn`.
- **Invalid enum values**: unknown `status`/`priority` → skip or fall back to a neutral style; do not crash render.
- **Progress out of range**: clamp to 0–100 before setting width and `aria-valuenow`.
- **Accessibility contrast**: verify each status/priority color against card background at ≥ 4.5:1 text / ≥ 3:1 non-text.
- **Reduced motion**: no essential animation; respect `prefers-reduced-motion`.
- **File-scope overlap risk**: `app/app.js` is Coder-owned but referenced from Designer's `index.html`. Mitigated by freezing the script filename (`./app.js`) and template ids in Step 1.

## 10. Open questions

1. **Static server vs. `file://`?** `fetch()` of `./project-data.json` typically fails under `file://`. Should the launch config use a lightweight static server (deterministic port) plus a browser launch to `http://localhost:8080/index.html`, or is a simple-browser-opens-file config acceptable? Recommendation: static server.
2. **Filename of the render glue.** This plan assumes `app/app.js`. Confirm before Designer wires the `<script>` tag.
3. **JSON top-level shape.** Object `{ "projects": [...] }` (recommended) vs. bare array `[...]`. This plan assumes the object form.
4. **Error styling class.** Should the Designer add a dedicated `.dashboard__error` style, or should the render script reuse `.dashboard__empty`? Recommendation: dedicated class, still Designer-owned.
5. **Dark mode / theming.** In or out of scope for v1?
6. **Interactivity.** Any filtering/sorting by status or priority in v1, or purely read-only cards? This plan assumes read-only.
7. **Branding.** Any logo/wordmark for the header beyond the text "Project Pulse"?
