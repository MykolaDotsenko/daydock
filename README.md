# DayDock

**A workday planner for capture, priorities, focus, follow-ups and daily review — without productivity scores or streak pressure.**

[**Open the live app →**](https://mykoladotsenko.github.io/daydock/) · [Architecture](./ARCHITECTURE.md)

![Quality](https://github.com/MykolaDotsenko/daydock/actions/workflows/quality.yml/badge.svg)
![Deploy Pages](https://github.com/MykolaDotsenko/daydock/actions/workflows/pages.yml/badge.svg)

![DayDock Today view showing the current Now outcome, Focus controls, and Top 3 priorities](./docs/assets/daydock-today.webp)

## Product loop

```text
capture → decide → focus → follow up → review
```

DayDock is designed around reducing repeated decisions during the day rather than storing an endless backlog.

## What it does

- **Top 3 + Now** — keep daily priorities small and make one outcome explicitly current;
- **Later scheduling** — Tomorrow, Next week, custom date and Someday;
- **Recurring work** — Daily, Weekdays, Weekly and Monthly recurrence;
- **Focus Mode** — pause/resume/complete with timestamp-derived timing;
- **Calendar Awareness** — import a local `.ics` snapshot and derive busy/focus windows;
- **People follow-ups** — lightweight person/task links and follow-up dates;
- **Command Palette** — search tasks and people with `Ctrl/Cmd + K`;
- **Backup/restore + Undo** — recover from destructive actions and move data between browsers;
- **PWA** — offline shell, installability, share target and optional notifications;
- **Cross-tab sync** — workspace changes stay ordered across open tabs.

There is no account or application backend.

## State model

One canonical workspace drives the UI.

```text
React features
      ↓
useSyncExternalStore
      ↓
DayDock store
      ├── versioned persistence
      ├── BroadcastChannel sync
      ├── backup / restore
      └── calendar boundary
      ↓
pure domain reducer
      ├── tasks + recurrence
      ├── Top 3 / Now
      ├── focus lifecycle
      └── people / follow-ups
```

The reducer does not create IDs, read the clock or access browser APIs. Those values enter from the application boundary so domain transitions remain deterministic.

Persisted/imported data is validated before it becomes canonical state.

## A few implementation choices

### Focus timing

Canonical focus state stores timestamps rather than decrementing seconds every tick. The displayed timer is derived from those timestamps, so tab throttling does not corrupt elapsed time.

### Recurrence

Completing recurring work records the completion and derives the next instance instead of rewriting history.

### Cross-tab ordering

BroadcastChannel messages are validated and use deterministic ordering so two open tabs do not blindly overwrite each other.

### Calendar import

Calendar Awareness parses a local `.ics` snapshot and normalizes recurrence behind its own boundary. It is intentionally not presented as a live calendar connection.

## Stack

- React 19
- TypeScript 6 strict
- Zod 4
- Vite 8
- CSS Modules / modern CSS
- Web Storage
- BroadcastChannel
- Service Worker / Web App Manifest
- Vitest + Testing Library
- Playwright
- ESLint
- GitHub Actions + GitHub Pages

Runtime dependencies are React, React DOM and Zod.

## Quality

```bash
npm ci
npm run check
npm run build:pages
```

Browser release tests cover desktop/mobile, focus flow, recovery and offline reopening.

The Pages build also verifies that generated assets stay under the repository deployment path instead of silently breaking after a rename/base-path change.

Production Pages deploys only after **Visual Smoke** succeeds on `main`. The deployment checks out that verified commit SHA, re-runs the application verification, and then builds the Pages artifact.

## Run locally

Requires Node.js 24+.

```bash
npm ci
npm run dev
```

## Quick technical review

- [`src/domain/daydock/reducer.ts`](./src/domain/daydock/reducer.ts) — deterministic transitions
- [`src/storage/dayDockPersistence.ts`](./src/storage/dayDockPersistence.ts) — validation/migration
- [`src/store/synchronizedDayDockStore.ts`](./src/store/synchronizedDayDockStore.ts) — cross-tab sync
- [`src/domain/calendar/ics.ts`](./src/domain/calendar/ics.ts) — calendar parsing
- [`src/features/focus/FocusMode.tsx`](./src/features/focus/FocusMode.tsx) — focus workflow
- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — boundaries and trade-offs

## Privacy and limits

Workspace and imported calendar data stay in the browser by default.

Browser notifications are best-effort and are not described as guaranteed background delivery.

No open-source license is claimed unless one is explicitly added to the repository.
