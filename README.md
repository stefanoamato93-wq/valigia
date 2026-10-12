# Valigia

A mobile-first packing checklist for frequent travel. Single self-contained
`index.html` (no build step, no backend). Seeded from the Google packing sheet.

Live: https://stefanoamato93-wq.github.io/valigia/

## What it does

- **Categorised checklist** of everything to pack, grouped into categories
  (Viaggio, Vestiti, Bagno, Tech, Accessori by default).
- **Tap an item** (circle or label) to mark it packed / unpacked. Packed items
  get a check and are greyed out with a strike-through.
- **Progress** card at the top shows `X / Y in valigia`, a percentage and a
  progress bar (turns green at 100%).
- **Metti tutto** marks every item packed. **Svuota** empties the suitcase
  (marks everything unpacked) to reset for the next trip. Both are undoable.
- **Da mettere** filter (next to the search box) hides packed items and
  fully-packed categories, so you only see what's left.
- **Search** finds any item fast, lets you toggle it packed, or create a brand
  new one (`+ Crea`) into a category you pick.
- **Add** items with the floating `+` (pick a category) or inline at the bottom
  of each category.
- **Edit mode** (gear, top right) reveals rename (✎) and delete (🗑) buttons on
  every item and category, plus **+ Nuova categoria**.
- **Expand / collapse** all categories from the list header.
- **Undo toast** on every destructive action (delete item, delete category,
  Metti tutto, Svuota).

The five requested actions map as: **add** (+ FAB / inline / search Crea),
**remove** (edit mode 🗑 or search 🗑), **modify** (edit mode ✎ rename),
**mark all as complete** (Metti tutto), **empty** (Svuota).

## Data model

Keyed objects with stable IDs (so single fields can be rewritten without
clobbering the whole list):

```
state = {
  cats: {
    <catId>: {
      name, order, collapsed,
      items: { <itemId>: { name, order, packed } }
    }
  }
}
```

- `packed` (boolean) is the only per-item state. Default seed is everything
  `packed: false`.
- Persisted to `localStorage` under key `valigia_v1`. The "Da mettere" filter
  state is stored separately under `valigia_filter_v1`.
- First load with no saved state seeds from the `DEFAULT` object (the list taken
  from the Google sheet). After that, the saved state wins.

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

- **Change the default packing list:** edit the `DEFAULT` object at the top of
  the `<script>` in `index.html`. Only affects a fresh browser (existing saved
  state is not overwritten); clear `localStorage` key `valigia_v1` to re-seed.
- **Rename categories / add categories:** do it in the app (edit mode), or edit
  `DEFAULT`.
- **Colours:** `--brand` / `--brand-dark` CSS vars for the theme; per-category
  accent colours come from `categoryPalette`.
- **Change the storage key:** `KEY` / `FILTER_KEY` constants.

## Deploy

Push to `main`; GitHub Pages serves the root. See the workspace `my-apps`
steering for the push + Pages routine.
