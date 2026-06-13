# AB Event Planner

A board to plan graduation/festival events. Each event starts with topics for
**School Play, Content (Visuals & Audio), Kid Training, Clothes, and Props** — and you can
add, rename, reorder, or remove topics per event. Every topic has a topic lead and tasks,
every task can have **subtasks**, and Content tasks additionally get **Visuals/Audio**
subtask panes. Plus a **"By person"** view, **undo**, **dark mode**, and **online sharing**.

It's a single self-contained `index.html` — no build, no dependencies.

## Use it

- **Online:** https://eventorgo.netlify.app/
- **Locally:** open `index.html` in your browser. (Note: **online sharing only works over the
  web** — the hosted link above, or any web host — not when opening the file directly. Local
  folder-saving works fine from the file.)

## Saving your work

Your board is saved automatically in your browser on whatever device you use it. For something
more durable, pick one of:

- **📂 Connect project folder** (Chrome/Edge) — saves the board straight into a folder on your
  computer (`data/board.json`), with timestamped restore points in `history/`. Reconnects
  automatically next time. Fully private, no internet needed.
- **🔗 Share online** — puts the board on the internet so you (and others) can reach it from
  anywhere. See [Sharing](#sharing-with-the-team) below.
- **⬇ Save backup** / **⬆ Restore** — download or load a dated `.json` copy at any time, in any
  browser. A good habit regardless of which mode you use.

## Editing

Click almost any text to edit it — event name, date, topic name, team leads, tasks. Tick
checkboxes as work gets done; progress rolls up per topic and per event.

- **+ Add event**, **+ Add topic**, **+ add task / subtask** to build out the board.
- **↶ Undo** reverses the last few destructive actions (delete task/topic/event, reset).
- **reset this event** restores one event's topics/tasks to the starting template.
- Every delete asks for confirmation first.

## Sharing with the team

Click **🔗 Share online** → leave the box blank → it creates an online board and gives you a
**board ID**. Share that ID with others:

- **Viewers** paste the ID under 🔗 Share online → they see the board read-only (no password,
  no account). They can expand tasks to view subtasks too.
- **Editors** click **✏️ Edit mode** and enter the password (default `123456`) → they can edit.
- **🔑 Change password** sets a new edit password (recommended); **🗑 Delete board** removes the
  shared board entirely.

Saving a shared board is **manual** (to stay within the free online quota): click **💾 Save
online**, or it saves automatically when you return to **All events**. You'll be warned if you
try to close the tab with unsaved changes.

See **SHARING.md** for the full walkthrough.

> The edit password is a convenience lock — it's stored with the board and checked in the
> browser, so it prevents accidents but isn't real security. Don't put anything sensitive in a
> shared board.

## Source & deployment

This is a git repository hosted at https://github.com/Ammarjeddin/EventOrg. Pushing to `master`
auto-deploys to Netlify. The runtime files `data/board.json` and `history/*.json` are git-ignored
(they're your live data, not source).
