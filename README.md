# EventOrg — Graduation & Festival Command Center

A board to manage your graduation/festival events. Each event starts with topics for
School Play, Content (Visuals & Audio), Kid Training, Clothes, and Props — and you can
add, rename, reorder, or remove topics per event. Each topic has a topic lead, tasks,
and (for Content) Visuals/Audio subtasks, plus a "By person" view, undo, and dark mode.

## Files

- `index.html` — the app. This is the whole thing; open it to use the board.
- `data/board.json` — your live data. Written automatically once you connect the folder.
- `history/` — timestamped restore points (a new one at most every couple of minutes while you work).
- `README.md` — this file.

## How to open it

Double-click `index.html` and it opens in your browser.

For data to save **into this folder on disk**, use **Chrome** or **Edge**
(other browsers can't write to folders yet — see fallback below).

## Saving your work (so it's never lost again)

1. Open `index.html` in Chrome/Edge.
2. Click **📂 Connect project folder** in the top bar.
3. Pick this `EventOrg` folder and allow access.
4. The status line turns green: **✔ Saved to folder**. From now on, every change
   you make is written to `data/board.json` automatically, and restore points are
   saved into `history/`.

Next time you open the board it reconnects to the folder (you may get a quick
one-click permission prompt).

### Always-available backups

Regardless of browser, the top bar also has:

- **⬇ Save backup** — downloads a dated `.json` copy of everything.
- **⬆ Restore** — loads a backup file back in.
- **🛟 Recover** — pulls whatever is still saved in the browser and downloads a copy.

If you're not using Chrome/Edge, just use **Save backup** regularly and keep the
files in this folder.

## Version history with git

This folder is a git repository, so you can keep a full timeline of the board.
After a working session, save a snapshot from a terminal in this folder:

```
git add -A
git commit -m "Update board after Monday's event"
```

Each commit is a point you can always return to. The `history/` folder also keeps
automatic JSON restore points even if you never touch git.

## Sharing with the team (later)

Today this is single-user (data lives in this folder / your browser). When you want
real-time shared editing with view-only and admin roles, the next step is putting
`data/board.json` on a shared backend (Firebase/Supabase) or moving to Airtable/
Google Sheets. Ask and we'll set it up.
