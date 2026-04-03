# Salla Twilight — custom home component pattern (Theme Raed)

This document describes the **repeatable workflow** used for the first custom editable component (**Announcement bar**, path `home.announcement-bar`). Follow the same steps for new home blocks.

---

## 1. What “custom editable component” means here

- **Editable** = merchant configures fields in the **theme / home page builder** (Twilight), not only in code.
- **Home component** = block rendered inside `{% component home %}` on the storefront index; each block maps to a Twig template and a JSON schema in `twilight.json`.
- This repo extends **Theme Raed**; conventions (`s-block`, `component.*`, `public/` build) stay the same.

---

## 2. File placement

| Piece | Location | Notes |
|--------|-----------|--------|
| Twig template | `src/views/components/home/<name>.twig` | File name = last segment of the path (`home.<name>` → `<name>.twig`). |
| Component schema | `twilight.json` → `components[]` | One object per custom block; see §3. |
| Feature flag | `twilight.json` → `features[]` | Add `component-<kebab-name>` so the theme declares support (aligned with other `component-*` entries). |
| Styles | Prefer `src/assets/styles/04-components/home-blocks.scss` | Scope under `.s-block--<kebab-block>`; matches existing home blocks. |
| JS (optional) | `src/assets/js/home.js` or a partial | Only if the block needs behavior beyond CSS. Load via `index.twig` `{% block scripts %}` — already pulls `home.js`. |
| Built assets | `public/app.css`, etc. | Run `pnpm run prod` after SCSS/JS changes; commit `public/` outputs. |

**Do not** add Flowbite or ad-hoc globals; stay consistent with Raed + Tailwind + existing SCSS structure.

---

## 3. Schema / settings (`twilight.json`)

Each custom home block is one object in the **`components`** array with:

| Field | Purpose |
|-------|---------|
| `key` | Stable UUID (unique per component). |
| `title` | `{ "en": "...", "ar": "..." }` — labels in the home page builder. |
| `icon` | Salla icon class, e.g. `sicon-format-text-alt` (match other components). |
| `path` | **`home.<segment>`** — must match Twig path: `home.announcement-bar` → `announcement-bar.twig`. |
| `fields` | Array of field definitions the editor shows (boolean, string, collection, `variable-list`, etc.). |

Additionally:

- **`features`**: include **`component-<kebab-name>`** (e.g. `component-announcement-bar`) next to other `component-*` features so Partners / theme capabilities stay in sync.

**Field IDs and Twig:**

- Collection subfields use dotted IDs, e.g. `announcements.text`, `announcements.url`.
- In Twig, rows are usually exposed as **`component.<collectionId>`** (e.g. `component.announcements`) with each item using **short keys** (`item.text`, `item.url`), matching other Raed components (see `custom-testimonials.twig`, `main-links.twig`).
- Booleans / scalars: **`component.<fieldId>`** (e.g. `component.is_visible`).

**Reference implementations in this repo:**

- Same file: `twilight.json` — Announcement bar entry (collection + `variable-list` for optional URL).
- Existing patterns: **Brands**, **Main links**, **Custom testimonials** — copy field shapes (icons, `sources` for links, `multilanguage`, `minLength` / `maxLength`).

**Not used for this pattern:**

- Top-level **`settings`** in `twilight.json` — those are **global theme options** (`theme.settings`), not per–home-block. Use `components` + `fields` for builder blocks.

---

## 4. Rendering (Twig)

1. **Docblock** at top of the Twig file: document `component`, `component.*`, and `position` (Salla provides **`position`** for ordering on the home page).
2. **Guard clauses**: if the block should not output empty markup, check visibility flags and that collections have usable rows (as in `announcement-bar.twig`).
3. **Root wrapper**: use **`s-block s-block--<name>`** for consistency; inner classes can be BEM-style (`announcement-bar__*`).
4. **Data**: read only from **`component`** (and global objects like `store` / `theme` if needed). Do not assume undeclared fields.

Home page entry point:

- `src/views/pages/index.twig` → `{% component home %}` — no change per component; new blocks appear once registered and added in the builder.

---

## 5. Preview integration (Salla CLI / Partners)

1. **Build**: `pnpm install` then `pnpm run prod` so `public/` reflects SCSS/JS.
2. **Push** theme repo (branch Partners uses, e.g. `master`).
3. **Partners**: theme linked to GitHub; preview uses deployed theme + **Twilight** home layout.
4. **Merchant step**: add the block in the **home page builder** — blocks are **not** auto-inserted; order and visibility are configured there.
5. **Feature list**: `features` in `twilight.json` drives what the theme **advertises**; custom components still need to be **placed** on the home layout in the editor.

If something does not show: confirm the commit is the one the store imports, hard-refresh preview, and that the block was added to the home layout.

---

## 6. Checklist — new custom home component

1. [ ] Add `component-<kebab-name>` to `features` in `twilight.json`.
2. [ ] Append a new object to `components` with unique `key`, bilingual `title`, `path`: `home.<segment>`, and `fields`.
3. [ ] Create `src/views/components/home/<segment>.twig` with `component` + `position` docblock and markup.
4. [ ] Add styles under `.s-block--<kebab-name>` in `home-blocks.scss` (or a partial imported from `app.scss` if the team splits files later).
5. [ ] Add JS only if required; wire through existing home bundle if possible.
6. [ ] Run `pnpm run prod`; commit source + `public/` changes.
7. [ ] Push; in Partners, add block to home layout and test preview.

---

## 7. Announcement bar — reference mapping

| Concept | Value |
|---------|--------|
| Feature | `component-announcement-bar` |
| Path | `home.announcement-bar` |
| Twig | `src/views/components/home/announcement-bar.twig` |
| Editor title | EN: “Announcement bar”, AR: “شريط الإعلانات” |
| Styles | `home-blocks.scss` → `.s-block--announcement-bar` |

Use this row as a template row in §6 for the next component.
