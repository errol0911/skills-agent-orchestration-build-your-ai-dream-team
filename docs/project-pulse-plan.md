# Project Pulse Implementation Plan

## Summary

Mona's team needs **Project Pulse**: a small static dashboard that shows each project's name, owner, status, recent activity, priority and a short summary for contributors. It should be a polished, card-based, responsive page with status badges.

Deliverables (source: `.github/project-pulse-brief.md`, `.github/steps/3-step.md`):

| File | Purpose | Owner |
| --- | --- | --- |
| `app/project-data.json` | Dashboard data, top-level `projects` array | **Coder** |
| `app/index.html` | Page markup plus an inline script that fetches `project-data.json` and renders `.project-card` elements | **Coder** |
| `app/styles.css` | All visual design: layout, cards, badges, responsive behavior | **Designer** |
| `.vscode/launch.json` | "Run Project Pulse Dashboard" launch config that serves `app/` and opens `index.html` | **Coder** |

Each file has exactly one owner. No other agent edits a file it doesn't own. Fixes go back to the owner through the Orchestrator.

The team:
- **Orchestrator** coordinates and doesn't implement.
- **Planner** wrote this plan.
- **Coder** (GPT-5.5) owns the implementation files and runnable-app support.
- **Designer** (Gemini 3.7 Flash) owns UI/UX and styling.

No agent stages, commits or pushes. The learner does that.

## Shared contracts (frozen before implementation)

These contracts are the main dependency between the files. Because they're fixed here, Designer and Coder can work in parallel without editing each other's files.

### Data schema contract: `app/project-data.json`

- The file is strict JSON (no comments, no trailing commas) with a top-level object containing a `projects` array.
- Required fields on every project (all strings): `name`, `owner`, `status`, `recentActivity`, `priority`.
- Optional fields: `summary` (contributor-friendly one-liner), `lastUpdated` (ISO date `YYYY-MM-DD`), `id` (kebab-case, unique).
- Allowed `status` values (lowercase, kebab-case): `on-track`, `at-risk`, `blocked`, `completed`.
- Allowed `priority` values: `high`, `medium`, `low`.
- Include 4–6 sample projects. Together they must cover every status value and every priority value so every badge style is visible.
- Field names are case-sensitive and must match exactly. The Step 3 workflow checks for `projects`, `name`, `owner`, `status`, `recentActivity` and `priority`.

### Class-name contract: `app/index.html` markup ↔ `app/styles.css` selectors

| Class | Element / meaning |
| --- | --- |
| `.dashboard` | Root `<main>` wrapper for the whole dashboard (**required by grader**) |
| `.dashboard-header` | Header holding the `<h1>Project Pulse</h1>` title and subtitle |
| `.dashboard-summary` | Optional row of counts by status (for example "2 at risk") |
| `.project-grid` | Container (`<section>` / `<ul>`) for the cards; responsive grid |
| `.project-card` | One card per project, rendered as `<article>` or `<li>` (**required by grader**) |
| `.project-card__header` | Card top row: name and status badge |
| `.project-name` | Project name (`<h2>`) |
| `.project-owner` | Owner line |
| `.project-activity` | Recent activity text |
| `.project-summary` | Optional summary text |
| `.project-updated` | Optional last-updated date (`<time>`) |
| `.status-badge` + `.status-badge--{status}` | Status pill. Modifiers: `--on-track`, `--at-risk`, `--blocked`, `--completed`, `--unknown` |
| `.priority-badge` + `.priority-badge--{priority}` | Priority marker. Modifiers: `--high`, `--medium`, `--low`, `--unknown` |
| `.dashboard-message` (+ `--error`, `--empty`, `--loading`) | Status/feedback region for loading, empty and error states |

Rule: Coder outputs only these class names, and Designer styles only these class names. If a new class is needed, the Orchestrator adds it to this table first.

### Launch contract: `.vscode/launch.json`

The file must be strict JSON with no comments. It contains one configuration with these values:

| Key | Value |
| --- | --- |
| `name` | `Run Project Pulse Dashboard` (exact) |
| `type` | `node-terminal` (built into VS Code, so no extra extension; the Python extension isn't in `.devcontainer/devcontainer.json`) |
| `request` | `launch` |
| `command` | `python3 -m http.server 5500` |
| `cwd` | `${workspaceFolder}/app` (serve from the app directory) |
| `serverReadyAction.pattern` | A regex that captures the port from Python's `Serving HTTP on ... port 5500` line, e.g. `Serving HTTP on .* port ([0-9]+)` |
| `serverReadyAction.uriFormat` | `http://localhost:%s/index.html` (opens the dashboard, not a directory listing) |
| `serverReadyAction.action` | `openExternally` |

The top-level `version` is `0.2.0`.

## Ordered implementation steps

### Phase 0: Plan approval (sequential; Orchestrator)
1. The Orchestrator reviews this plan, confirms the contracts above, and resolves or defers the open questions.

### Phase 1: Build (parallel; Coder and Designer have disjoint files)

**Coder track (sequential within the track):**
1. Create `app/project-data.json` following the data schema contract.
2. Create `app/index.html`:
   - `<!doctype html>`, `<html lang="en">`, `<meta charset="utf-8">`, `<meta name="viewport" content="width=device-width, initial-scale=1">`.
   - `<title>Project Pulse</title>` and a visible `<h1>Project Pulse</h1>` (exact title).
   - `<link rel="stylesheet" href="styles.css">` (relative path).
   - `<main class="dashboard">` containing the header, an optional summary, a `.dashboard-message` region with `role="status"` / `aria-live="polite"`, and the `.project-grid` container.
   - An inline `<script>` that fetches `project-data.json` (relative URL) and renders one `.project-card` per project. It shows `status`, `recentActivity` and `priority` as visible text, and the source must contain those literal identifiers because the grader checks for them.
   - Build the DOM with `textContent`/`createElement`, never `innerHTML` with data values.
   - Map `status`/`priority` to the BEM modifier classes. Any value outside the allowed list maps to `--unknown` and still shows the raw text.
   - Handle the loading, error and empty states (see Edge cases).
3. Create `.vscode/launch.json` following the launch contract.

**Designer track:**
1. Create `app/styles.css`, styling only the class-name contract:
   - `.dashboard` layout: max width, centered, comfortable padding, clear header hierarchy.
   - `.project-grid`: CSS Grid with `repeat(auto-fill, minmax(~280px, 1fr))` and a gap, so it reflows to 1 column on phones and 2–4 on wider screens.
   - `.project-card`: `border-radius`, `box-shadow`, padding, background contrast and a hover/focus-within lift. The **`border-radius` and `box-shadow` properties are checked by the grader.**
   - `.status-badge--*`: pill shape with distinct colors. Badges always carry their text label, so the design doesn't rely on color alone.
   - `.priority-badge--*`: strong treatment for `high` (for example an accent border or bold label), softer for `medium`/`low`.
   - `.dashboard-message--*` styles for the error, empty and loading states.
   - Typography: system font stack, readable line length and line height.
   - Use CSS custom properties for colors and spacing.
   - Add `:focus-visible` outlines and a `prefers-reduced-motion` guard on transitions.
   - Add a media query or fluid units so content is readable at 360px.
2. Report design decisions, tradeoffs and recommended checks to the Orchestrator.

### Phase 2: Integration and validation (sequential; Orchestrator, with fixes routed to file owners)
1. Run the validation checklist below.
2. Send any defect to the owning agent only: markup or script goes to Coder, visuals go to Designer, data and launch config go to Coder.
3. Re-validate, then hand off to the learner, who runs **Run Project Pulse Dashboard** and commits.

## Designer responsibilities

- Owns **`app/styles.css`** exclusively.
- Delivers a polished dashboard, not a bare page: project cards, status badges, clear priority treatment, readable spacing and a responsive layout.
- Includes the required CSS hooks `.dashboard` and `.project-card`, plus `border-radius` and `box-shadow`.
- Owns the visual hierarchy: title → summary → cards. Inside each card: name and status first, then owner, activity and priority.
- Accessibility: text contrast of at least 4.5:1 (including badge text on badge backgrounds), visible focus styles, no meaning conveyed by color alone, and support for reduced motion.
- May recommend markup changes (for example a better heading level) to the Orchestrator, but does **not** edit `app/index.html`.

## Coder responsibilities

- Owns **`app/index.html`**, **`app/project-data.json`** and **`.vscode/launch.json`** exclusively.
- Implements the data schema and class-name contracts exactly.
- Writes clear, explicit rendering code with informative error messages. It shouldn't rely on build tools or external libraries.
- Creates `.vscode/launch.json` as **strict JSON with no comments**, with `cwd` = `${workspaceFolder}/app`, opening `http://localhost:%s/index.html` so learners see the dashboard, not a directory listing.
- Validates its files (JSON parse, local serve) before reporting.
- Does **not** edit `app/styles.css`. If the markup needs no styles, it relies on the Designer's selectors.

## Dependencies

1. **Data schema → `index.html`**: the renderer reads `projects[].name/owner/status/recentActivity/priority`. Renaming a field breaks rendering.
2. **Data enums → CSS modifiers**: `status`/`priority` values become `.status-badge--{value}` and `.priority-badge--{value}`. The enum lists and the CSS modifiers must match exactly.
3. **Class-name contract → `index.html` and `styles.css`**: markup and CSS must use the same names, or cards render unstyled.
4. **`index.html` → relative paths**: `styles.css` and `project-data.json` load relative to `app/`. That only works if the server's root is `app/`, which `launch.json` sets via `cwd`.
5. **`launch.json` → `index.html`**: `uriFormat` targets `/index.html`, so `app/index.html` must exist at that path.
6. **Phase 2 depends on all of Phase 1** being complete.

## Parallel work decisions

- **Can run in parallel:** the Coder track (`app/project-data.json`, `app/index.html`, `.vscode/launch.json`) and the Designer track (`app/styles.css`).
  - The file scopes don't overlap.
  - The data dependency between them (class names and enum values) is settled by the frozen contracts above, so neither track needs the other's output to start.
- **Must run sequentially:**
  - Phase 0 comes before Phase 1, because the contracts must be approved before anyone builds against them.
  - Within the Coder track, `project-data.json` comes before `index.html`, so the renderer is written and tested against real data. This is the same owner working in order, not a cross-agent block.
  - Phase 2 comes after Phase 1, because integration checks need all four files.
  - Fix loops in Phase 2 are sequential per file owner, to avoid two agents editing the same file.
- **Fallback:** if the Orchestrator changes any contract mid-build, pause the affected track, update the contract here, then resume. If parallel output drifts, run Designer after Coder so it can style the real markup.

## Edge cases

| Case | Expected handling |
| --- | --- |
| Opened via `file://` (double-clicking `index.html`) | `fetch` fails (browsers block or reject file URLs). Catch the error and show a `.dashboard-message--error` that says to use **Run Project Pulse Dashboard** or `python3 -m http.server` from `app/`. |
| `project-data.json` missing / 404 | Check `response.ok`. Show an error message that includes the HTTP status. |
| Invalid JSON | Catch the JSON parse failure and show an error message. Log details to the console. |
| `projects` missing or not an array | Treat it as a data error and show a clear message. |
| Empty `projects` array | Show `.dashboard-message--empty` ("No projects to show yet"). Don't show an empty grid with no explanation. |
| Unknown `status` / `priority` value | Use the `--unknown` modifier (neutral style) and still display the raw value. |
| Missing optional fields (`summary`, `lastUpdated`) | Omit those elements. Don't render "undefined". |
| Missing required field on one project | Render the card with a visible placeholder such as "Not set" for that field, and warn in the console. Don't fail the whole page. |
| Long names / long activity text | CSS handles overflow (`overflow-wrap: anywhere`) so cards keep the layout intact. |
| Data containing HTML characters | Rendering via `textContent` prevents injection. |
| Port 5500 already in use | The server fails to start, so `serverReadyAction` never fires. Note this in the handoff: stop the old server or change the port. |
| Codespaces browser preview | `localhost:5500` is auto-forwarded by Codespaces; `openExternally` opens the forwarded URL. |
| Caching after edits | A hard refresh may be needed. Low risk; no special handling needed. |

## Validation expectations

Automated / command-line validation (Orchestrator or Coder runs it):
1. `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` both succeed. The Step 3 workflow runs both commands.
2. Grader keyphrase checks (mirroring `.github/workflows/3-step.yml`):
   - `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`.
   - `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`.
   - `app/project-data.json` contains `projects`, `name`, `owner`, `status`, `recentActivity`, `priority`.
   - `.vscode/launch.json` contains `Run Project Pulse Dashboard` and `index.html`.
3. A Python one-liner (or the Coder) confirms that every project has all 5 required fields and that the status and priority values are in the allowed lists.
4. Served check: from `app/`, run `python3 -m http.server 5500`. Then `curl -sf http://localhost:5500/index.html` and `curl -sf http://localhost:5500/project-data.json` both return 200. Stop the server afterwards.
5. Don't use `scripts/validate-exercise.sh` as a gate. It's a template-maintenance script and intentionally fails once learner files such as these are tracked.

Manual / visual validation (learner or Orchestrator):
6. Run and Debug → **Run Project Pulse Dashboard** opens `/index.html`, showing the dashboard and not a directory listing.
7. One `.project-card` appears per project, each showing name, owner, status badge, recent activity and priority.
8. Responsive check at about 360px, 768px and 1280px widths: 1 column on phones, multiple columns on wide screens, no horizontal scrolling.
9. Accessibility basics:
   - `lang="en"`, a single `<h1>`, landmarks (`<main>`), and a heading per card.
   - Badges have text labels.
   - Contrast of at least 4.5:1.
   - Keyboard focus is visible.
   - The message region is announced (`aria-live`).
10. Error-path check: open `app/index.html` via `file://` (or temporarily rename the data file, without committing that change) and confirm the friendly error appears.

## Open questions

1. **Inline vs. external script:** the four assigned files don't include a JS file, so the renderer goes inline in `index.html`. Is a separate `app/app.js` acceptable? The current recommendation is no, to keep file ownership to the four listed files.
2. **`file://` fallback:** should `index.html` also embed fallback data so it works without a server? Recommendation: no, so `project-data.json` stays the single source of truth; show the instructional error instead.
3. **Launch type:** `node-terminal` is recommended because it needs no extension. If the learner has the Python extension, `debugpy` with `module: http.server` also works. Confirm which the Orchestrator prefers.
4. **Status vocabulary:** are `on-track / at-risk / blocked / completed` and `high / medium / low` the right terms for Mona's team, or should "risk" be a separate field from priority? The brief says "priority or risk level"; this plan uses `priority` only.
5. **Sample data content:** should project names and owners be fictional placeholders or real team members? This plan assumes fictional examples (for example owner `@mona`).
6. **Sorting/filtering:** out of scope for this version. Should cards be sorted, for example by priority then status? This plan uses data-file order.
