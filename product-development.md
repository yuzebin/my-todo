# Product Development Document — my-todo

## 1. Product Summary
**my-todo** is a lightweight daily to‑do list app. The core experience is to quickly create tasks each day, then mark them done, edit them inline, or delete them. The visual theme uses **blue / green** as the primary colors. The implementation language is **TypeScript**, and deployment target is **Vercel**.

## 2. Problem Statement
Many people keep daily tasks in scattered places (notes, chats, paper). They need a fast, low‑friction way to:
- Capture tasks as they arise
- Review what they planned “today”
- Update tasks as priorities change
- Finish the day with a clear sense of completion

## 3. Goals and Non‑Goals
### Goals (MVP)
- Provide a simple “Today” task list with: **Add**, **Edit**, **Delete**, **Done**
- Make editing extremely fast: click an item to edit inline
- Persist tasks reliably so a refresh does not lose data
- Support daily organization (today vs other days)
- Ship a production deployment on Vercel

### Non‑Goals (MVP)
- Team collaboration and shared lists
- Complex project management (epics, dependencies, Gantt)
- Full calendar/task scheduling system
- Advanced access control (unless added later)

## 4. Target Users
- Individuals who want a personal daily task tracker
- Students tracking homework/errands day‑by‑day
- Developers/knowledge workers who prefer a minimalist UI over feature‑heavy apps

## 5. Core Use Cases (User Stories)
- As a user, I can add a new task in one action so I don’t forget it.
- As a user, I can click a task to edit it so I can quickly fix wording or update details.
- As a user, I can mark a task done so I can track progress.
- As a user, I can delete a task so I can clean up irrelevant items.
- As a user, I can switch days (or review previous days) to see what I planned and what I completed.

## 6. Scope and Requirements

### 6.1 Functional Requirements (MVP)
1. **Daily List**
   - A “Today” view shows tasks for the current local date.
   - Option A (simple): tasks belong only to the current date.
   - Option B (recommended): user can navigate to past days to review history.

2. **Add Task**
   - User can add a task via a button and/or enter key.
   - A task minimally contains: title, created time, done flag.
   - Validation: prevent empty titles; trim whitespace.

3. **Edit Task (Click-to-Edit)**
   - Clicking a task title enters edit mode inline.
   - Save on Enter / blur; cancel on Escape.
   - Editing preserves done state.

4. **Done / Undone**
   - A task can be toggled done.
   - Done tasks are visually distinct (e.g., muted text + strikethrough).

5. **Delete**
   - A task can be deleted.
   - Deleting should not be easily accidental:
     - Option A: immediate delete with undo toast
     - Option B: confirm dialog

6. **Persistence**
   - Tasks persist across refreshes and browser restarts.
   - MVP storage options:
     - Local-only: localStorage or IndexedDB
     - Cloud: a backend store (adds auth + database)

### 6.2 Nice-to-Have (Post‑MVP)
- Search and filter (done/undone)
- Drag to reorder tasks
- Recurring tasks
- Tags / priority levels
- “Archive” completed tasks automatically after N days
- Export (JSON/CSV)
- Sync across devices (requires authentication + backend)

## 7. UX and Information Architecture

### 7.1 Primary Screens
1. **Today**
   - Date header
   - Input/add control
   - Task list (undone first, done after)
2. **History** (if included in MVP)
   - Date picker / calendar list
   - Displays tasks for selected date
3. **Settings** (optional)
   - Theme preferences (blue/green palettes)
   - Data export/import

### 7.2 Key Interaction Details
- Add flow:
  1) Type title → 2) Press Enter or click Add → 3) Task appears at top or bottom (choose one)
- Edit flow:
  1) Click title → 2) Inline input shows existing value → 3) Enter saves, Esc cancels
- Done flow:
  - Toggle without opening edit mode (separate checkbox/button area)

## 8. Visual Design Guidelines
- Primary colors: **Blue** + **Green**
- Accessibility:
  - Ensure color contrast meets WCAG AA where possible
  - Do not rely on color alone to indicate done state (also use shape/strikethrough/icon)
- Typography:
  - Clear hierarchy: date header > input > task list

## 9. Data Model (Suggested)

### 9.1 Task
- `id: string` (uuid or timestamp-based)
- `date: string` (YYYY-MM-DD, local date key)
- `title: string`
- `done: boolean`
- `createdAt: number` (ms since epoch)
- `updatedAt: number` (ms since epoch)

### 9.2 Storage Schema (Local)
- Key by date:
  - `tasksByDate: Record<string, Task[]>`
  - or flat array with date index

### 9.3 Migration and Versioning
- Store a `schemaVersion` with persisted data to allow future upgrades (e.g., tags, priority).

## 10. Edge Cases and Rules
- Editing a done task should be allowed.
- If a user deletes a task, prefer an undo mechanism to prevent accidental loss.
- If the user changes system date/timezone:
  - Use local date at runtime; tasks keyed by YYYY‑MM‑DD.
  - Document that “Today” is computed from local time.
- Deduping: allow duplicate titles; identity is by `id`.

## 11. Non‑Functional Requirements
- Performance: handle at least 500 tasks/day without visible lag.
- Reliability: persistence writes are atomic (avoid partial writes).
- Security & Privacy:
  - MVP local-only: no server collection; data stays in browser storage.
  - If adding backend later: use standard auth, protect user data, avoid logging sensitive content.
- Cross-browser support: modern evergreen browsers.
- Offline: local-only mode should work offline.

## 12. Metrics (What to Measure)
### Product Metrics
- DAU/WAU (if analytics enabled)
- Tasks created per day
- Completion rate (% tasks marked done)
- Retention: returning users (1d/7d)

### UX Metrics
- Median time to add a task (from page load)
- Edit success rate (edit initiated vs saved)
- Error rate (validation failures)

## 13. Technical Plan (Recommended Baseline)

### 13.1 Frontend
- TypeScript throughout
- UI framework recommendation (choose based on repo setup):
  - React + Next.js (pairs well with Vercel)
  - Or Vite + React (also deployable to Vercel)

### 13.2 State + Persistence
- In-memory state for UI responsiveness
- Persist on each change:
  - localStorage (simple, fast enough for MVP)
  - IndexedDB (better for larger data; slightly more complexity)

### 13.3 Testing (MVP)
- Unit tests for reducers/storage helpers (create/edit/delete/toggle)
- Basic e2e smoke test: load → add → edit → done → delete

### 13.4 Deployment
- Vercel deployment pipeline
- Environments:
  - Preview deployments per PR
  - Production deployment on main branch

## 14. Delivery Plan (Milestones)
1. **MVP UI + CRUD**
   - Today view, add/edit/delete/done
2. **Persistence**
   - Store tasks locally; load on startup
3. **Daily Organization**
   - Date keying; optional history navigation
4. **Polish**
   - Blue/green theme refinement, accessibility pass
5. **Release**
   - Production deploy on Vercel; README update with usage

## 15. Acceptance Criteria (MVP)
- User can add a non-empty task; it appears immediately.
- Clicking a task title enters edit mode; Enter saves; Esc cancels.
- User can toggle done/undone; done tasks render distinctly.
- User can delete a task; deletion is reflected immediately and persisted.
- Refreshing the page preserves tasks and states for the current day.
- Visual theme uses blue/green as primary accents and remains readable.

## 16. Open Questions (To Finalize Before Build)
- Should the MVP include **history navigation** (previous days), or only “Today”?
- Should delete require confirmation, or provide an undo toast?
- Should tasks be ordered by creation time, manual order, or allow drag‑reorder?
- Is cross-device sync required in v1 (implies auth + backend), or is local-only acceptable?

