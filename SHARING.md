# Sharing a board online

EventOrg can share a board over the internet so others can view (and, with a password, edit) it.
It uses a free online JSON store (JSONBin) built into the app — **no accounts, no setup, no Google.**

> **Security note:** the edit password is a *convenience lock*. It keeps people from accidentally
> changing things, but it's stored with the board and checked in the browser — it is **not** real
> security. Don't put anything sensitive in a shared board.

## Create a shared board

1. Build your board as usual.
2. Click **🔗 Share online** → leave the box blank → **OK**.
3. The app creates an online board and shows you a **board ID** (a 24-character code).
4. Share that **board ID** with others. The default edit password is **`123456`**.
5. **Change the password** (recommended): while editing, click **🔑 Change password**, enter a new one
   and re-type it to confirm. It takes effect for everyone immediately — share the new password with
   your co-editors.

## Open / view a board someone shared

1. Click **🔗 Share online**, paste the **board ID** (or a link containing it), **OK**.
2. You see the board **read-only**.
3. To edit, click **✏️ Edit mode** and enter the password (default `123456`).

## Saving (manual, on purpose)

To stay within the free online quota, shared boards **do not auto-save on every change**. Instead:

- While editing, a **💾 Save online** button appears. It shows an asterisk (`Save online*`) when you
  have unsaved changes.
- Your work is saved to the online board when you **click 💾 Save online**, or automatically when you
  go back to **▦ All events**.
- If you try to **close the tab with unsaved changes**, the browser warns you first.

## Notes

- One board ID = one shared board. Everyone using the same ID sees the same board.
- Changes you save are visible to others the next time they open/reconnect the board ID.
- Prefer keeping data on your own machine? Use **📂 Connect project folder** instead — that saves the
  board to a folder on your computer, no internet needed.
