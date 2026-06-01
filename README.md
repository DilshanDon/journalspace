# JournalSpace — Persistent Notes & Journal App

> **Assessment V3-c · Persistent Mini-App**
> Author: Dilshan | 3rd Year CS Student

---

## What it does

JournalSpace is a browser-based personal notes and journal manager. You can create, read, update, and delete journal entries. Each entry has a title, content, category, and tags. Everything is saved to `localStorage` so your data survives browser closes and restarts — no server needed.

**Extra feature beyond CRUD:** real-time full-text search combined with tag filtering. You can search across title, body, tags, and category simultaneously, and click any tag on a card to instantly filter all entries by that tag. Both filters work together.

---

## How to Run

No installation required. No npm. No build step.

### Option 1 — Open directly (simplest)

```bash
# macOS
open app.html

# Windows
start app.html

# Linux
xdg-open app.html
```

### Option 2 — Local server (recommended, avoids browser `file://` restrictions)

**Python (comes pre-installed on most machines):**
```bash
python3 -m http.server 8080
```
Then open: [http://localhost:8080/app.html](http://localhost:8080/app.html)

**Node.js (if installed):**
```bash
npx serve .
```
Then open the URL it prints.

---

## Verifying Persistence

1. Open the app and create 2–3 entries
2. Close the browser tab completely
3. Reopen `app.html`
4. Your entries will still be there

Data is stored in `localStorage` under the key `journalspace_dilshan_v1`.

---

## Features

| Feature | Detail |
|---|---|
| **Create** | Add entries with title, content, category, tags |
| **Read** | Card grid view, sorted by newest by default |
| **Update** | Edit any entry via the pencil icon |
| **Delete** | Remove entries with a confirmation prompt |
| **Search** | Real-time search across all fields |
| **Tag filter** | Click any tag to filter by it; click again to clear |
| **Category filter** | Dropdown populated from your own categories |
| **Sort** | Newest / Oldest / A→Z |
| **Persistence** | `localStorage` — survives browser restarts |

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| `Ctrl + N` / `Cmd + N` | New entry |
| `Esc` | Close modal |

---

## File Structure

```
.
├── app.html      ← entire application (single file, no dependencies)
├── README.md     ← this file
└── ANSWERS.md    ← assessment Q&A
```

---

## Tech

- Vanilla HTML, CSS, JavaScript — no frameworks, no build tools
- `localStorage` API for persistence
- `crypto.randomUUID()` for unique entry IDs
- Google Fonts (Syne + DM Mono) via CDN
