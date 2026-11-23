# Project Pulse final handoff

## Final result

Project Pulse is a static, card-based dashboard for Mona's team. It shows each project's name, owner, status, recent activity and priority, plus an optional summary and last-updated date. It loads 5 sample projects from `app/project-data.json`. Together they cover every status (`on-track`, `at-risk`, `blocked`, `completed`) and every priority (`high`, `medium`, `low`). The page has a status summary line, status and priority badges, a responsive grid, dark mode, and friendly loading, empty and error messages.

| File | Purpose | Owner |
| --- | --- | --- |
| `app/index.html` | Markup plus an inline script that fetches the data and renders `.project-card` elements with `textContent` | Coder |
| `app/styles.css` | Layout, cards, badges, responsive, dark mode, focus and reduced-motion styles | Designer |
| `app/project-data.json` | Top-level `projects` array (5 projects) | Coder |
| `.vscode/launch.json` | **Run Project Pulse Dashboard** launch configuration | Coder |
| `docs/agent-team.md` | Agent roster, models and workflow | Learner (with Copilot CLI) |
| `docs/project-pulse-plan.md` | Implementation plan and the shared contracts | Planner (saved by Coder) |

## Agent team and contributions

| Agent | Model | What it did |
| --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Coordinated the work in phases: plan, then a parallel Coder/Designer build with disjoint file scopes, then integration and validation. It routed fixes to the owner of each file and did not implement anything itself. |
| Planner | Claude Opus 5.5 (copilot) | Wrote `docs/project-pulse-plan.md`: the data schema, class-name and launch contracts, phases, edge cases, validation steps and open questions. Planner has no edit tool, so Coder saved the plan to disk. |
| Designer | Gemini 3.7 Flash (copilot) | Created `app/styles.css`. After the integration review, it added a `[hidden] { display: none !important; }` rule and removed `text-transform: capitalize` from the badges. |
| Coder | GPT-5.5 (copilot) | Created `app/project-data.json`, `app/index.html` and `.vscode/launch.json`. |

Here's how the plan was used. Because the frozen contracts (data schema, class names, launch config) were agreed up front, Coder and Designer could build in parallel without touching each other's files. Phase 2 checked the result against those same contracts.

## How to run

- **VS Code:** Open Run and Debug, then choose **Run Project Pulse Dashboard**. This runs `python3 -m http.server 5500` with `cwd` `${workspaceFolder}/app` and opens `http://localhost:5500/index.html`. Codespaces forwards the port automatically.
- **Terminal alternative:** `cd app && python3 -m http.server 5500`, then open `http://localhost:5500/index.html`.

## Dashboard validation

These results come from running the checks locally on 2026-11-23.

| Check | Result |
| --- | --- |
| `python3 -m json.tool app/project-data.json` | PASS |
| `python3 -m json.tool .vscode/launch.json` | PASS |
| Schema: top-level `projects` list; every project has non-empty string `name`, `owner`, `status`, `recentActivity`, `priority`; status and priority values are in the allowed sets | PASS (5 projects; all 4 statuses and all 3 priorities present) |
| `.vscode/launch.json`: name is exactly "Run Project Pulse Dashboard", command `python3 -m http.server 5500`, cwd `${workspaceFolder}/app`, uriFormat `http://localhost:%s/index.html` | PASS |
| `app/index.html` keyphrases: `<title>Project Pulse</title>`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` | PASS |
| `app/styles.css` keyphrases: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `@media` | PASS |
| Class contract: every class `app/index.html` can emit (26, including all `--{status}`, `--{priority}`, `--unknown` and message modifiers) has a selector in `app/styles.css` | PASS |
| Inline script extracted from the page and piped to `node --check -` (no temp files) | PASS |
| Served `app/` on port 5500 and curled `/index.html`, `/styles.css`, `/project-data.json` | PASS (200 / 200 / 200) |
| Server stopped; port 5500 free afterwards | PASS |
| Step 3 workflow checks (`.github/workflows/3-step.yml`): keyphrases in all four files plus both `json.tool` runs | PASS (run locally) |
| Step 4 workflow checks (`.github/workflows/4-step.yml`): keyphrases in this file | PASS (see the grep check reported with this handoff) |

**Not verified** (no browser or assistive-technology tooling in this run):

- How the page actually renders in a browser.
- Responsive layout at specific widths.
- Dark mode appearance.
- Behavior of the `file://` error path in a real browser.
- Keyboard focus visibility.
- Screen reader announcements.
- Color contrast ratios.

**Inconsistencies found between the plan and the implementation** (all minor, none blocking):

- The `file://` error message suggests `http://localhost:8000/index.html`, which is the default port for a bare `python3 -m http.server`. The launch config uses port 5500. Both are correct, but they differ.
- On cards that have a summary, `.project-summary` sits between the owner and the recent activity. The plan's order was owner → activity → priority, with the summary optional and its position unspecified.
- `.priority-badge--high` uses `text-transform: uppercase` on purpose (Designer's emphasis choice). Only `capitalize` was removed.

## Learner handoff

1. **Open the dashboard:** Run and Debug → **Run Project Pulse Dashboard**. Check that the dashboard opens, not a directory listing, and that there are 5 cards.
2. **Manual checks to do in the browser:**
   - Resize to about 360px, 768px and 1280px wide. You should see 1 column on narrow screens, more on wide ones, and no horizontal scrolling.
   - Tab through the page and check that keyboard focus is always visible.
   - Open `app/index.html` directly (`file://`) and confirm the friendly error message appears.
   - Switch the OS or browser to dark mode and check readability and badge contrast.
3. **Known risks:**
   - If port 5500 is already in use, the server won't start and the browser won't open. Stop the old server (`kill <PID>`) or change the port in `.vscode/launch.json`.
   - Opening the page via `file://` shows a friendly error by design, because the data is fetched. The error text points to port 8000, the default for `python3 -m http.server`.
   - `:has()` (status accent bar, high-priority card border) and `color-mix()` (tints and focus rings) are progressive enhancements. Older browsers fall back to plain cards and badges that stay readable.
4. **Open questions from the plan, still deferred:**
   - Inline script vs. a separate `app/app.js` (inline was kept).
   - Embedded fallback data for `file://` (not added).
   - `node-terminal` vs. `debugpy` launch type (`node-terminal` was kept).
   - Status and priority vocabulary, and whether "risk" should be its own field.
   - Fictional vs. real sample owners (fictional handles were used).
   - Sorting and filtering (data-file order is used).
5. **Git:** No agent staged, committed or pushed anything. You control all git operations through Copilot CLI prompts. Next, stage `docs/final-handoff.md`, commit it with "Validate and hand off Project Pulse", and push.
