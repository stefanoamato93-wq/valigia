# Valigia

A mobile-first packing checklist. Single self-contained `index.html` (no build
step, no backend). Two independent lists live as tabs: **Travel** (seeded from
the Google packing sheet) and **Gym**.

Live: https://stefanoamato93-wq.github.io/valigia/

## What it does

- **Two tabs — Travel and Gym** in the sticky header. Each tab is its own
  independent checklist (own categories, items, packed state and completion %).
  The active tab is remembered between sessions.
- **Categorised checklist** of everything to pack, grouped into categories
  (Travel: Viaggio, Vestiti, Bagno, Tech, Accessori; Gym: In palestra, Per
  doccia, Vestiti, Addosso, Zaino). Items inside each category are always shown
  in alphabetical order (default sort).
- **Tap an item** (circle or label) to mark it packed / unpacked. Packed items
  get a check and are greyed out with a strike-through.
- **Progress** card at the top shows `X / Y in valigia`, a percentage and a
  progress bar (turns green at 100%).
- **Always-visible completion %** in the sticky header: a percentage pill plus a
  thin progress bar sit in the top bar, so the overall % stays in view while you
  scroll the list.
- **Metti tutto** marks every item packed. **Svuota** empties the suitcase
  (marks everything unpacked) to reset for the next trip. Both are undoable.
- **Mark a whole category** packed or empty: each category header has a round
  check toggle. If the category is not fully packed, tapping it packs the whole
  category; if it is already full, tapping empties it. Undoable. (Always visible,
  no need to enter edit mode.)
- **Da mettere** filter (next to the search box) hides packed items and
  fully-packed categories, so you only see what's left.
- **Search** finds any item fast, lets you toggle it packed, or create a brand
  new one (`+ Crea`) into a category you pick.
- **Add** items with the floating `+` (pick a category) or inline at the bottom
  of each category.
- **Edit mode** (gear, top right) reveals move (⇄), rename (✎) and delete (🗑)
  buttons on every item; on each category it shows reorder arrows (↑ / ↓),
  rename and delete, plus **+ Nuova categoria**.
- **Reorder categories:** in edit mode use the ↑ / ↓ arrows on a category header
  to move it up or down. The order is saved and persists. (Items are not manually
  orderable; they stay alphabetical.)
- **Move items between categories**, single or in bulk:
  - *Single:* in edit mode tap the ⇄ button on an item and pick the destination
    category.
  - *Bulk:* tap **Sposta** in the list header to enter selection mode. Tap any
    items (across categories) to select them, or use **Tutti** in a category
    header to grab the whole category. The bottom bar shows the count and
    **Sposta in…** opens the destination picker; **Fine** exits. Moving one or
    many lands them in the chosen category.
- **Expand / collapse** all categories from the list header.
- **Undo toast** on every destructive or bulk action (delete item, delete
  category, Metti tutto, Svuota, move single/bulk, mark category packed/empty).

The five requested actions map as: **add** (+ FAB / inline / search Crea),
**remove** (edit mode 🗑 or search 🗑), **modify** (edit mode ✎ rename),
**mark all as complete** (Metti tutto), **empty** (Svuota). Plus **move** items
between categories (edit-mode ⇄ for one, **Sposta** selection mode for bulk).

## Data model

Keyed objects with stable IDs (so single fields can be rewritten without
clobbering the whole list). The top level holds one list per tab:

```
allData = {
  active: "travel" | "gym",
  lists: {
    travel: { cats: { <catId>: { name, order, collapsed,
                                 items: { <itemId>: { name, order, packed } } } } },
    gym:    { cats: { ... same shape ... } }
  }
}
```

`state` always points at `allData.lists[activeId]`, so every operation below
works on `state.cats` exactly as in the single-list version. Switching tabs
re-points `state` and re-renders. Tabs are a fixed set (`TAB_DEFS`): Travel and
Gym.

- `packed` (boolean) is the only per-item state. Default seed is everything
  `packed: false`.
- Item display order is **always alphabetical** (`itemList` sorts by `name`,
  Italian locale, case-insensitive, `numeric` on so "2a chiave" sorts naturally).
  The per-item `order` field is still stored but no longer drives item display.
- Category display order follows the category `order` field, which the user
  controls with the ↑ / ↓ arrows in edit mode. `moveCatOrder(cid, dir)` swaps a
  category with its neighbour and reassigns sequential `order` values (0..n-1).
- `markCategory(cid, val)` sets every item in one category to `packed = val`
  (used by the per-category round toggle) and registers one undo. `renderProgress`
  updates both the top progress card and the sticky-header pill/bar (`#hPct`,
  `#hBar`) from `totals()`.
- Moving an item keeps its stable `<itemId>` and `packed` state; it just
  relocates the entry from the source category's `items` to the destination's
  (with a fresh `order` at the end). `moveItems(targets, destCid)` handles one or
  many and registers a single undo that restores each item's original category
  and order.
- Persisted to `localStorage` under key `valigia_v2` (the whole `allData`). The
  "Da mettere" filter state is stored separately under `valigia_filter_v1` and is
  shared across tabs.
- **Migration:** on first run with no `valigia_v2`, the app reads the old
  single-list key `valigia_v1` and makes it the **Travel** tab (so existing data
  carries over untouched), then adds a fresh **Gym** tab. If there is no old data
  either, Travel seeds from `DEFAULT` and Gym from `DEFAULT_GYM`. `valigia_v1` is
  left in place, not deleted.

## Storage / sync

Data lives in the browser's `localStorage`, so it is **per-device** and works
fully offline. There is no login and nothing leaves the device.

Cross-device sync is **not** enabled yet. The groceries app syncs via its
Firebase Realtime Database, but that database's security rules are locked to its
own room, so this app cannot write to it. To add sync later, either (a) open the
groceries Firebase project and add a rule allowing a `valigia` node, or (b)
create a dedicated Firebase project and drop its config in. The data model is
already shaped for a keyed Realtime-DB write path.

## Install icon (pin / add to home screen)

The app ships a full icon set so it shows a proper suitcase icon when pinned or
installed from Chrome, and when added to the iOS home screen:

- `favicon.svg` - vector tab/pin icon (source of truth for the artwork: brand
  square `#0f6e8c` + white suitcase glyph).
- `icon-192.png`, `icon-512.png` - PNG icons referenced by the web app manifest
  (declared `any` and `maskable`; the full-bleed brand background is
  maskable-safe).
- `icon-180.png` - `apple-touch-icon` for iOS home screen.
- `manifest.webmanifest` - PWA manifest (name, `standalone` display,
  `theme_color`/`background_color` `#0f6e8c`, icon list).

The PNGs are rendered from `favicon.svg` at each size. To regenerate after
changing the artwork: open the SVG in a browser (or a temp HTML page that
scales it to the viewport), set the viewport to 512/192/180 and screenshot each,
or use any SVG-to-PNG tool. Keep the glyph inside the centre ~80% so the
maskable crop never clips it.

## Common edits

- **Change the default lists:** edit `DEFAULT` (Travel) or `DEFAULT_GYM` (Gym) at
  the top of the `<script>` in `index.html`. Defaults only apply on a fresh
  browser (existing saved state is not overwritten); clear `localStorage` keys
  `valigia_v2` (and `valigia_v1`) to re-seed.
- **Add another tab:** add an entry to `TAB_DEFS` (`{ id, label }`), add a
  matching default object, and seed/normalise it in `loadAll` / `normalizeAll`
  the same way `gym` is handled.
- **Rename categories / add categories:** do it in the app (edit mode), or edit
  the relevant `DEFAULT*` object.
- **Colours:** `--brand` / `--brand-dark` CSS vars for the theme; per-category
  accent colours come from `categoryPalette`.
- **Change the storage key:** `KEY` / `OLD_KEY` / `FILTER_KEY` constants.

## Deploy

Push to `main`; GitHub Pages serves the root. See the workspace `my-apps`
steering for the push + Pages routine.
