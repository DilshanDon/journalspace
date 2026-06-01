# ANSWERS.md

---

## 1. How to run

The whole app is a single HTML file with no dependencies.

**Quickest way:**
```bash
# Just open it in your browser
open app.html        # macOS
start app.html       # Windows
xdg-open app.html    # Linux
```

**Or serve it locally (avoids any `file://` quirks):**
```bash
python3 -m http.server 8080
# Then go to: http://localhost:8080/app.html
```

Nothing to install. No Node, no npm, no config. The only external resource is Google Fonts, which loads from a CDN on first visit and gets cached automatically — the app still functions fully without it, just with fallback fonts.

To test persistence: add a few entries, close the tab entirely, reopen `app.html` — everything should still be there.

---

## 2. Stack choice

**I chose: plain HTML + CSS + Vanilla JavaScript, with `localStorage` for persistence.**

I'm a 3rd year CS student and I've been through React, databases, all of it in coursework. My honest reason for not using any of that here: the task doesn't need it. A single-file HTML app that you open in a browser and it just works is the cleanest possible solution for a single-user persistent mini-app. No build step, no package manager, nothing to install on a fresh machine.

For storage, `localStorage` is the right call at this scale. It's synchronous, zero-config, and persists across sessions. I considered SQLite with a small Python/Flask backend for a second, but that means running two processes, dealing with file paths, and making the "how to run" instructions a paragraph long. Not worth it when `localStorage` already does exactly what the spec asks.

**What would have been a worse choice: React + a hosted database (Firebase or Supabase).**

I actually started sketching this in React because I use it a lot in coursework, and I caught myself setting up `useState`, `useEffect`, an API layer — for what? A notes app one person uses. That's over-engineering. React's component model is genuinely useful when your UI has complex state interactions across many components. A single-page notes app with a modal form doesn't hit that threshold. It would have also added a `node_modules` folder and a build step, which makes "run on a fresh machine" a real chore.

---

## 3. One real edge case

**Edge case: corrupted or unparseable data in `localStorage`**

**File:** `app.html`
**Function:** `loadNotes()` — roughly line 130 in the script block

```js
function loadNotes() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
  } catch (e) {
    console.warn('JournalSpace: failed to parse stored data, starting fresh.', e);
    return [];
  }
}
```

**What the edge case is:**
`localStorage` stores everything as a raw string. If that string is ever invalid JSON — because the browser cut off a write mid-session (e.g. the tab crashed while saving), the user edited storage manually in DevTools, or a browser extension touched the key — `JSON.parse()` throws a `SyntaxError`.

**What happens without the handling:**
The error propagates unhandled. The `notes` variable never gets assigned. Every subsequent call that reads `notes` (rendering the grid, updating stats, filtering) either crashes or reads `undefined`. The user sees a completely blank page with no explanation.

**What happens with the handling:**
The `catch` block logs a warning to the console and returns `[]`. The app boots into a clean empty state — the grid renders correctly, the stats show zero, and the user can start adding entries again. They lose the corrupted data, but the app stays usable. Without this, one bad write would brick the entire app until the user manually cleared their browser storage.

---

## 4. AI usage

I used **Claude (Anthropic)** while building this.

| Where I used it | What I asked | What it gave me | What I changed |
|---|---|---|---|
| Initial structure | Asked it to sketch a vanilla JS CRUD app with localStorage | A working scaffold with `loadNotes`, `saveNotes`, a render loop, and a basic modal | I rewrote the modal open/close logic — it was using `display:block/none` with inline styles, which I swapped to a CSS class toggle for cleaner separation |
| Styling | Asked for a dark UI that doesn't look generic | CSS with custom properties, card animations, and the green accent palette | Adjusted the colour values — the original green was too neon and felt off. I dialled it back to something easier on the eyes for a journal app |
| Search feature | Asked how to filter notes by a search query in real time | A `getFiltered()` function that matched against title and body | Extended it myself to also search tags and category, since those felt obvious to include |
| Tag filter | Asked it to add tag-click filtering | It added `activeTag` state and toggled it on click | The bug I found: clicking a tag while a category filter was already active would leave both applied with no way to clear the tag independently. I rewrote `clearFilters()` to reset all three (search input, category dropdown, and `activeTag`) together, and added the "Clear filters" button that only appears when at least one filter is active |

**The most significant thing I changed:** the original `clearFilters` only reset the search input. So if you filtered by a tag, then typed in the search box, then cleared — the tag filter stayed silently active. You'd see fewer results than expected with no obvious reason why. I caught this while testing and fixed it so all filters reset together.

---

## 5. Honest gap

**The app has no data export or backup.**

Right now everything lives in one browser's `localStorage`. That means:
- Clearing browser data (history, cookies, site data) silently wipes all your entries with no warning
- You can't access your notes from another browser or device
- There's no recovery path if storage gets corrupted

**What I'd do with another day:**

Add a simple **Export to JSON** button. It's maybe 15 lines of code: `JSON.stringify` the notes array, create a `Blob`, trigger a `<a download>` click. Then an **Import from JSON** button that uses `FileReader` to read the file back and merge with existing entries (deduplicating by ID). No server, no accounts, just a file the user keeps on their own machine.

That would turn the current "hope your browser doesn't clear data" situation into something genuinely usable as a real personal journal. I'd also add a warning before the delete button that tells you how many entries you have, so clearing feels deliberate rather than accidental.
