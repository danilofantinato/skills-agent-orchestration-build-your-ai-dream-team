# Project Pulse Dashboard: Implementation Plan

## 1. Goal and scope
**Goal:** Build a static HTML/CSS/JS dashboard that shows project health data for Mona's projects, with no build step and no backend.

**In scope:** project cards with status badges (on-track, at-risk, off-track) and priority treatment; a summary header with counts per status; data loaded from `app/project-data.json`; responsive layout and accessible markup; a VS Code launch config.

**Out of scope:** frameworks, bundlers, backend, authentication, persistence, git operations.

## 2. File assignments

| File | Owner | Purpose | Action |
|---|---|---|---|
| `app/project-data.json` | Coder | Data schema and sample data | Create |
| `app/index.html` | Coder | Semantic markup, inline JS that loads and renders data, agreed class hooks | Create |
| `app/styles.css` | Designer | All visual design, layout, tokens, responsive rules, focus/contrast styling | Create |
| `.vscode/launch.json` | Coder | Launch config with cwd `${workspaceFolder}/app` that opens `index.html` | Create |
| `docs/project-pulse-plan.md` | Planner | This plan | Create |
| `.vscode/tasks.json`, `docs/agent-team.md`, `.github/agents/*` | Nobody | Out of scope | Do not touch |

JS stays inline in `index.html` by default.

## 3. Responsibilities

### Designer (owns `app/styles.css` and design direction)
- Design tokens as CSS custom properties (`--color-ok`, `--color-risk`, `--color-bad`, spacing, radius, font stack).
- Responsive grid for `.dashboard` using `auto-fit/minmax`, working from 320px to desktop.
- Style `.project-card`, status badges and priority treatment.
- Accessibility: text contrast of at least 4.5:1; status never conveyed by colour alone (text label plus optional icon/shape); visible `:focus-visible`; `prefers-reduced-motion`; optional dark mode.
- Hand off a class and token contract (section 4).

### Coder (owns markup, JS, data, launch config)
- **Data file:** `project-data.json` with 5-8 varied projects covering every status and priority, including at least one edge case. Schema: `{ "updated": ISO date, "projects": [{ "id", "name", "owner", "status": "on-track|at-risk|off-track", "priority": "high|medium|low", "progress": 0-100, "summary", "dueDate" }] }`.
- **Markup:** semantic HTML with `<main>`, heading hierarchy, `lang`, viewport meta, `<link rel="stylesheet" href="styles.css">`. Cards are `<article>` elements in a list.
- **Loading/rendering:** `fetch('project-data.json')`; render with `textContent`, not `innerHTML`; handle loading, empty and error states via an `aria-live` / `role="alert"` region.
- **Launch config:** cwd `${workspaceFolder}/app`, opens `index.html`. `fetch` fails over `file://`, so serve over HTTP (e.g. a local server preLaunchTask plus Chrome/Edge launch).
- The Coder must not edit `styles.css`.

## 4. Dependencies
1. **Data schema -> rendering:** `index.html` rendering depends on field names and enum values in `project-data.json`. Freeze the schema first.
2. **Class and token contract -> both files:**
   - `.dashboard`, `.dashboard__summary`, `.project-card`, `.project-card__title`, `.project-card__meta`
   - `.status-badge` with modifiers `--on-track`, `--at-risk`, `--off-track`; `.priority--high|medium|low`
   - `.progress`, `.state-message` (loading, empty, error)
3. **Status enum -> CSS:** the Coder emits `status-badge--<status>` from JSON values; the Designer styles the same strings.
4. **Launch config** is independent but needed for final validation.
5. **Final integration** needs everything finished.

## 5. Parallel work decisions

**Phase 0 (sequential, Orchestrator):** fix the schema and class/token contract and include them in both delegation prompts.

**Phase 1 (parallel):**
- Coder: `app/project-data.json` and `.vscode/launch.json`.
- Designer: `app/styles.css`, written against the contract.
- Why parallel: file scopes do not overlap and both depend only on the Phase 0 contract.

**Phase 2 (sequential, after Phase 1 data):** Coder creates `app/index.html`. It needs the finalised data and contract, and links the existing CSS. Starting it in Phase 1 is possible if the schema is trusted, at the risk of rework.

**Phase 3 (sequential):** Designer review pass of rendered markup against CSS, editing only `styles.css`. Markup changes are reported to the Coder, so two agents never edit one file.

**Phase 4 (sequential, Orchestrator):** integrated verification and report to the learner.

## 6. Validation expectations
- **Launch:** use the launch config to open the app over HTTP with cwd `app/`; `index.html` loads.
- **Data:** every project renders as a card; all status and priority variants appear; a broken JSON path shows the error state (then restore).
- **Console:** no errors or warnings, no 404 for `styles.css`.
- **Responsive:** check ~320, 768 and 1280px; no horizontal scroll; grid reflows.
- **Accessibility:** heading order and landmarks; visible focus when tabbing; status shown by text, not just colour; contrast 4.5:1; `aria-live` regions work; optionally Lighthouse/axe.
- **Scopes:** read-only `git status`/`git diff` show changes only in assigned files; Coder did not touch `styles.css`; Designer did not touch JS or data; nothing staged or committed.

## 7. Edge cases
- Empty `projects` array; missing/null fields; `progress` outside 0-100.
- Unknown status falls back to a neutral badge.
- Very long names or summaries wrap or truncate; overdue dates.
- HTML in strings (safe via `textContent`).
- Fetch failure (`file://` CORS or 404); zero-count summary categories.

## 8. Open questions
- Which server does the launch config use (Live Server, `npx serve`, Python `http.server`)?
- Is inline JS acceptable, or should there be an `app/app.js`?
- What does "project health" include beyond status, priority and progress?
- Should sample data be fictional (assumed yes)?
- Is dark mode wanted?
