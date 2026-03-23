# Revamp: From Form to Wish List

## Overview

The current `commands.xforms.html` is a traditional HTML form — users fill in fields, select a command type, and submit. It works, but it doesn't feel natural. This update replaces it entirely with a **wish-based interface**: a small, personal automation hub where users create "wishes" — named automations that can be reviewed, tested, and executed on demand.

The redesign touches the UI, the mental model, the persistence layer, and the frontend stack.

---

## The Core Concept: Wishes

An automation is no longer a "command" — it's a **wish**. This shift in language sets the right tone: something the user wants to happen, described at a high level, executed when they're ready.

Each wish has:
- An **auto-generated name** derived from the command (e.g. "Move file → /photos")
- A **command type** (Move File, Email, or future types)
- The **configuration** for that command (paths, recipients, etc.)
- An **enabled/disabled state**
- An **execution log** (history of runs)

---

## Homepage

### First-time (empty state)
When no wishes exist, the user sees:
- A short, friendly message explaining what the app is and what a wish is
- A single prominent **"Add a wish"** call-to-action

No clutter. No fields. Nothing to figure out.

### With wishes
Each wish appears as a **minimal card** on the homepage, showing:
- The auto-generated wish name
- The command type badge (e.g. "Move File" / "Email")

Cards are listed in creation order. The **"Add a wish"** button is always accessible (e.g. a persistent button at the top or bottom of the list).

---

## Creating a Wish

Tapping **"Add a wish"** opens a command picker — a grid of **visual cards**, one per supported command type:

| Card | Description |
|------|-------------|
| **Move File** | Move a file from one path to another |
| **Email** | Send an email to a recipient |
| *(more can be added)* | The layout is designed to accommodate new types |

After selecting a command type, the user is walked through the minimal required fields for that command (not a form dump — just what's needed, one section at a time). When done, the wish is saved and immediately appears on the homepage.

---

## Wish Detail — Side Panel

Clicking any wish card opens a **side panel** that slides in from the right. It shows the full configuration of the wish and exposes all actions:

| Action | Description |
|--------|-------------|
| **Edit** | Modify the wish's configuration |
| **Duplicate** | Clone the wish as a starting point for a new one |
| **Enable / Disable** | Toggle whether the wish is active |
| **Test Run** | Execute the wish in dry-run mode — no real side effects, but the full flow is exercised |
| **Execution Log** | A history of past runs (timestamp, mode, outcome) |
| **Delete** | Remove the wish permanently |

The panel closes by clicking outside it or via an explicit close button.

---

## Test Run (Dry Run)

From the detail panel, users can run a wish in **test mode**. This mirrors the existing `dryRun` flag in the current payload — the wish is sent to the n8n webhook with `dryRun: true`, meaning the automation runs through its logic but performs no real-world action (no files moved, no emails sent).

The result of the test is shown inline in the panel (success/failure, any output from n8n).

---

## Persistence

All wishes are stored in **`localStorage`** — browser-native, no dependencies, no backend. There is no account, no data sent anywhere except to the user's own n8n webhook.

This is a deliberate privacy decision: if the app ever becomes publicly available, it holds zero user data. Everything stays in the user's browser.

**Wish data shape (stored locally):**

```json
{
  "id": "uuid",
  "createdAt": "ISO timestamp",
  "enabled": true,
  "command": "move",
  "name": "Move file → /destination",
  "config": {
    "targetPath": "...",
    "destination": "...",
    "dryRun": false
  },
  "log": [
    { "timestamp": "...", "mode": "test", "status": "success" }
  ]
}
```

---

## n8n Webhook Configuration

The n8n webhook URL is no longer visible on the main screen. It lives in a **settings panel** (e.g. accessible via a gear icon). The user enters it once and it's saved to localStorage alongside the wishes. The main interface never surfaces it.

---

## Frontend Stack

The current implementation is a single static HTML file with vanilla JS. This update replaces it with a **React + Vite** web application — fully client-side, no server required.

React Native / Expo is not used here. Since the target is web only, react-native-web would add abstraction with no benefit. Standard React with web primitives is the right fit.

| Concern | Choice |
|---------|--------|
| Framework | **React 19** |
| Build tool | **Vite** |
| Routing | **React Router v7** (or TanStack Router) — lightweight client-side routing |
| Styling | CSS Modules or a small utility library (e.g. vanilla-extract) — no Tailwind |
| Language | TypeScript |
| Linting | ESLint 9 |
| Package manager | npm |
| Persistence | **`localStorage`** — simple, browser-native, zero dependencies |

The app is fully client-side. There is no backend service in this project — the only external integration is the user's own n8n webhook.

---

## What's Being Removed

| Current element | Fate |
|----------------|------|
| XForms markup (`<xf:*>`) | Removed entirely — was non-functional in browsers anyway |
| Raw JSON preview textarea | Removed from main UI (may appear in detail panel for debug purposes) |
| n8n webhook URL on main screen | Moved to settings |
| "Add to queue" / queue reorder UI | Replaced by the wish list on the homepage |
| "Clear" / "Submit" buttons | Replaced by per-wish actions in the detail panel |
| Dry Run checkbox in the main form | Replaced by the explicit "Test Run" action in the detail panel |

---

## What's Being Kept

- The **payload structure** sent to n8n stays the same — this is a UI change, not a contract change
- The **two existing command types** (Move File, Email) carry over with the same fields
- The **dry run concept** — just surfaced differently (as a dedicated "Test Run" action)
- The **file input + base64 encoding** for move commands

---

## Project Structure

```
src/
  main.tsx               # React entry point

  pages/
    Home.tsx             # Homepage — wish list + "Add a wish" button
    NewWish.tsx          # Command picker (card grid)
    Settings.tsx         # n8n webhook URL configuration

  components/
    WishCard.tsx         # Minimal card shown on homepage
    WishDetailPanel.tsx  # Side panel with all wish actions
    CommandPicker.tsx    # Grid of command type cards
    commands/
      MoveFileForm.tsx   # Config fields for Move File
      EmailForm.tsx      # Config fields for Email

  lib/
    storage.ts           # localStorage read/write helpers
    wishes.ts            # Wish CRUD logic
    webhook.ts           # n8n POST logic (with dryRun flag)
    types.ts             # Shared TypeScript types

index.html
vite.config.ts
tsconfig.json
package.json
```

---

## Open Questions

- [ ] Should "Edit" reuse the `CommandPicker` + form flow, or open fields inline in the detail panel?
- [ ] What does the execution log look like visually — a simple timestamped list, or something richer (status icon, expandable output)?
- [ ] Should disabled wishes be visually distinct on the homepage (greyed out, a muted badge)?
- [ ] Is there a maximum number of wishes, or is the list unbounded?
- [ ] If the app ever needs multi-device access, what's the preferred sync strategy — export/import file, or a lightweight backend?
