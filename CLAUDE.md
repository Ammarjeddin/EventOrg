# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

EventOrg is a single-user planning board for managing graduation/festival events (5 by default, but events and their topics are user-editable). The **entire application is `index.html`** — one self-contained file with inline CSS and vanilla JavaScript, no build step, no dependencies, no framework. To run it, open `index.html` in a browser (Chrome/Edge for on-disk saving).

There is no build, lint, or test tooling. Changes are made directly in `index.html` and verified by opening the file in a browser.

## Architecture

Everything lives in the `<script>` block of `index.html`. The shape of the data model and the three render functions are the things to understand before editing.

### Data model (`data`)
A single global `data` object: `{ teamNames: {...}, events: [...] }`.
- Each **event** has `{id, name, grade, date, streams, topics}`.
- **Topics are per-event.** `ev.topics` is an **ordered** array of `{key, name, color}` that defines which topics an event has and in what order (reorder/add/delete all mutate this array). `ev.streams` is an object keyed by those topic `key`s; each stream is `{lead, items}`. Always iterate topics via the `eventTopics(ev)` helper (it lazily back-fills the default 5 from `WS_META` for events that predate per-event topics) — **never iterate `WS_META` directly in render/stats/by-person code.** New custom topics get a generated key (`t<timestamp>`) and a color cycled from `TOPIC_COLORS`.
- `WS_META` is now only the **template/default** topic set (keys `play`, `content`, `train`, `cloth`, `prop`) and color palette — used by `freshTopics()`/`freshStreams()`/`defaultData()`/`migrate()`. `data.teamNames` is a legacy global rename map, still honored as a fallback during migration but superseded by per-event `topics[].name`.
- Most streams have flat `items` of `{t, done, who}`.
- **The `content` key is the special case**: it's rendered "wide", and each of its items additionally carries `vis` and `aud` subtask arrays (Visuals / Audio, defined in `CONTENT_CATS`). Subtasks are `{t, done, who}`. Only the topic whose key is literally `content` gets subtask UI; custom topics are plain task lists. The `_open` flag on a content item is in-memory expand/collapse state and is **stripped from everything persisted** by `cleanData()` (see Persistence).

Topic UI lives in `renderDetail`/`wireDetail`: rename writes to `topics[].name`; reorder (`data-tmove="key|top|up|down|bottom"`) and delete (`data-tdel`, **enabled only when the topic's stream has zero items**) mutate `ev.topics`/`ev.streams`; "+ Add topic" (`#addTopic`) appends an empty topic.

### Three views (`view.mode`)
- `renderOverview()` — event cards grid (default), sorted by `byDate` (blank/invalid dates sink to the bottom, not the top). `daysUntil` returns `null` for blank/invalid dates — callers must handle `null` (don't let it become `NaN`).
- `renderDetail()` — one event's topic streams (iterated via `eventTopics`); the `content` stream uses the expandable subtask UI.
- `renderPeople()` — `collectAssignments()` walks every event/topic/item/subtask into a per-person map plus an "unassigned" bucket.

The render pattern throughout: build an HTML string, set `innerHTML`, then a paired `wireDetail`-style function attaches event handlers using `data-*` attributes (e.g. `data-ws`, `data-tedit`, `data-subcheck="i|cat|sidx"`). The `"i|cat|sidx"` pipe-delimited encoding in `data-*` attributes is how nested indices are passed to handlers — preserve this when adding interactions. `inlineEdit()` is the shared click-to-edit-text helper. Always `esc()` user text when building HTML strings.

After any mutation, call `save()` then re-render the active view.

### Undo
`undoStack` (in-memory, max `UNDO_MAX`=3) holds snapshots of the board *before* destructive actions. Call `pushUndo(label)` at the **start** of any destructive/irreversible action (task/subtask/event/**topic** delete, per-event reset, restore/recover) before mutating `data`; the toolbar `#undoBtn` pops them via `doUndo()`. The stack is in-memory only (cleared on reload).

### Persistence (in `save()` / `load()`)
Everything persisted goes through `cleanData()` (a deep clone with transient `_open` flags stripped) and the shared `backupPayload()` (used by both disk save and "Save backup", so formats can't drift).
1. **localStorage** (`LS_KEY`) — primary store, plus a rolling 12-version history under `HIST_KEY`. `setItem` failures surface a warning when no folder is connected (instead of failing silently). A cross-tab `storage` listener warns when another tab saved a newer version.
2. **On-disk project folder** — File System Access API (Chrome/Edge only, gated by `fsSupported`). The folder handle is persisted in IndexedDB (`idbGet`/`idbSet`) so it auto-reconnects on load. `writeDisk(forceHistory)` is **serialized** via a `writing`/`writeQueued` mutex (no overlapping `createWritable` sessions); writes are debounced (`scheduleDiskSave`, 700ms) into `data/board.json`, with timestamped restore points into `history/` (throttled to once per 2 min, or forced before destructive actions). `autoReconnect` will **not** silently overwrite in-memory edits — if the user edited before reconnect finished (`dirtySinceLoad`), it confirms first.
3. **Shared online board** — JSONBin (`api.jsonbin.io/v3`), gated by a built-in `JSONBIN_KEY` (an Access Key, sent as `X-Access-Key`; public bins read with no key). A board ID = a JSONBin bin id holding `{password, data}`. `connectSheet` opens/creates a board (blank input → `createSharedBoard`); `enterEditMode(pwArg?)` checks the password (default `DEFAULT_PASSWORD` = "123456") to leave `readOnly`; `deleteBoard()` (🗑 Delete board, editor-only, double-confirmed) calls `deleteBin` then `forgetSheet`; state persists under `SHEET_KEY` and `autoReconnectSheet` restores it read-only on boot. **Saving is MANUAL** (to conserve the free quota): edits call `markUnsaved()` (sets `unsavedChanges`, flips the `#saveOnlineBtn`), and a write to the bin happens only on **💾 Save online** click, on **returning to the overview** (`overviewBtn`), and is guarded by a `beforeunload` warning. `save()` never writes to the bin directly. The `readOnly` flag is also the read-only/viewer gate (see below). API helpers: `readBin`/`createBin`/`updateBin`/`deleteBin`.
4. **Manual backup/restore** — download/upload JSON files. (There is no "Recover" button — it was removed.)

**Read-only / viewer gate:** the `readOnly` global (true while viewing a shared board without the password) hard-stops `save()` and toggles `body.readonly`, whose CSS hides every editing affordance (add/delete/reorder/inline-edit) and disables checkboxes. Naming note: the shared-board code uses `sheetState`/`connectSheet`/`writeSheet`-style names for historical reasons (it began as a Google Sheets integration) — it is now JSONBin, not Sheets.

`data/board.json` and `history/*.json` are runtime data written by the app, not source — `data/` and `history/` ship with only `.gitkeep` files.

### Schema migration
`migrate(d)` runs on every load and on every import. It is the upgrade path for old saved data — it merges legacy separate `vis`/`aud` streams into the unified `content` stream, backfills `vis`/`aud` subtask arrays, and **back-fills per-event `topics`** (the default 5, honoring any old `teamNames` renames) for events saved before topics were per-event. **When you change the data shape, update `migrate()`** so existing saved boards (on users' disks / localStorage) keep loading.

## Conventions

- App name is **AB Event Planner** (title + `<h1>`). Internal identifiers/keys (`LS_KEY` `gradFestivalBoard.v1`, the `app:"grad-festival-command-center"` backup tag) are intentionally unchanged for backward-compat with existing saved boards — don't rename them.
- Headings use the **Fraunces** display serif (loaded from Google Fonts, `font-display:swap` → falls back to Georgia/serif if the request fails); body stays the system sans stack.
- **Every delete action confirms** (task, subtask, topic, event, online board). Keep this invariant when adding new destructive actions.
- Mobile: a `@media (max-width:640px)` block makes the overview/streams single-column and full-width buttons; `@media (pointer:coarse)` enlarges tap targets; overview cards have a staggered `cardIn` reveal (respects `prefers-reduced-motion`). Test changes at ~390px width.
- Match the existing terse, single-line-function style and the `var(--token)` CSS custom properties (light values in `:root`, dark overrides in `body.dark`).
- Dates use ISO `YYYY-MM-DD` strings; `isoAddDays` anchors the default template to 2026-06-15.
- The default per-event template (`defaultTasks`) is intentionally minimal: **1 task per topic, no subtasks**. New events and per-event "reset this event" use it.
- Reset is **per-event only** ("reset this event" in the detail header) — there is no global reset-all button.
- Event/topic counts are **not** hardcoded in copy — the overview subtitle (`#sub`) and the by-person eyebrow derive counts from `data.events.length` at render time. Don't reintroduce literal "5 events / 5 teams" strings.
- Online sharing uses JSONBin with **one built-in Access Key shipped in the source** (`JSONBIN_KEY`). This is deliberate (all boards share one free quota so users need no account) and the key is non-sensitive by design. The edit password is a convenience lock checked client-side — never represent it as real security.
- Sharing setup for users is documented in `SHARING.md`.
- Local-only and folder modes remain the default; online sharing is opt-in via 🔗 Share online.
