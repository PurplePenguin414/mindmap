# Mind Map

A self-hosted, password-protected mind-mapping app.

Type a title to create a map — it becomes the center node. Branch off any
node, with no limit on depth or branch count. Drag nodes anywhere; nothing
auto-arranges, ever. Connect any two nodes to each other with a custom line
style, independent of the tree structure. Levels (distance from the center)
are color-coded so they're distinguishable at a glance. Descriptions stay
hidden until you click a node, or you can reveal all of them at once.
Undo, autosave, dark mode, and PDF export are built in.

## How to use it

- **Create a map**: from the maps list, type a title and click Create. That
  title becomes the center node.
- **Add a branch**: click a node, then "+ Branch" in the toolbar that
  appears above it. Give it a title and (optionally) a description.
- **Edit a node**: click it, then "Edit."
- **Move a node**: just drag it. Nothing snaps or rearranges other nodes.
- **Reveal a description**: double-click a node to show/hide its own
  description, or use "Show all descriptions" in the top bar to reveal
  every one at once.
- **Connect two nodes**: click 🔗 on a node's toolbar (or "Connect" in the
  top bar), then click the second node. Pick a line style and color for
  that connection — each connection can look different. Click an existing
  connection line to edit its style or delete it.
- **Delete a node**: click it, then "Delete." Its own children reattach to
  its parent — they are never deleted along with it and never left
  orphaned. (If you delete the center node itself, each of its direct
  children becomes its own new center — see note below.)
- **Undo**: the Undo button (or Ctrl+Z) steps back through your last 50
  changes — adding, editing, deleting, moving, and connections all count.
- **Export**: "Export PDF" opens your browser's print dialog with the map
  laid out to fit the page — choose "Save as PDF" there.
- **Autosave**: every change saves to the server immediately. There's no
  save button to forget to click.
- **Dark mode**: the 🌓 button in the top-right, remembered per browser.

## Installing it yourself

Requirements: Node.js 22 or newer (this app uses `better-sqlite3`, which
needs Node ≥22).

```bash
npm install
node scripts/set-password.js "your password here"
```

That prints a line like `APP_PASSWORD_HASH=$2b$12$...` — create a `.env`
file in this folder with that line plus:

```
PORT=3000
SESSION_SECRET=some-long-random-string
APP_PASSWORD_HASH=$2b$12$...   # from the command above
```

Then:

```bash
npm start
```

Open `http://localhost:3000` and log in with the password you chose.

Run the automated data-logic tests any time with `npm test`.

### Changing the password later

```bash
node scripts/set-password.js "new password"
```

Paste the new `APP_PASSWORD_HASH` into `.env` and restart the app.

## Deploying with Docker

This folder includes a `Dockerfile` and `docker-compose.yml`.

1. Copy the whole folder to your server.
2. Create `.env` there as described above (generate the password hash with
   `node scripts/set-password.js` — either locally first, or once inside
   the built container: `docker compose run --rm mindmap node
   scripts/set-password.js "your password"`).
3. Pick a free local port in `docker-compose.yml` (defaults to `3000`) and
   put a reverse proxy with a TLS cert in front of it if you want it
   reachable from outside your network — e.g. Nginx/Apache/Caddy plus
   Let's Encrypt (certbot), or a tunnel like Cloudflare Tunnel.
4. `docker compose build && docker compose up -d` (if your server only has
   the older Docker Compose v1, use `docker-compose` with a hyphen instead
   — same commands otherwise).

The SQLite database lives in `./db` on the host (mounted into the
container as a volume), so it survives rebuilds. Back that folder up.

## A judgment call worth knowing about

The most natural reading of "delete a node reattaches its children to its
parent" doesn't say what should happen if you delete the *center* node
itself — there's no parent for its children to reattach to. This app has
each of the deleted center's direct children become its own new center,
rather than deleting the whole map or picking one child arbitrarily. Undo
(Ctrl+Z) reverses it instantly if that's not what you wanted.
