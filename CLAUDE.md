# Meal Prep Hub

Single-file vanilla-JS web app (no build step, no framework) for planning slow-cooker
chicken meal-prep batches: macros, bulk-up levers, batch scaling, shopping list.

## Files

- **`meal-prep-hub.html`** — the source of truth. Edit this one.
- **`index.html`** — exact copy of the above, deployed via GitHub Pages. After every
  edit: `cp meal-prep-hub.html index.html`, then commit + push.
- **`meal-prep-hub.jsx`** — stale original React draft, superseded by the HTML file.
  Not in use; ignore unless told otherwise.
- **`.claude/launch.json`** — local preview server config (`python -m http.server 3000`).

## Deployment

Repo: `github.com/ddworaczyk23-stack/meal-prep-hub` (git remote already configured,
credential manager handles auth via browser popup if needed).
Live URL: `https://ddworaczyk23-stack.github.io/meal-prep-hub/` (GitHub Pages serves
`index.html` from `main`).

Workflow after any change:
```
cp meal-prep-hub.html index.html
git add index.html && git commit -m "..." && git push origin main
```

User has two bookmarks: the local `meal-prep-hub.html` file (laptop, offline) and the
GitHub Pages URL added to iPhone home screen. Both must point to working state.

## Verifying changes

Use the preview MCP tools (`preview_start` with config name `meal-prep-hub`, then
`preview_eval` to navigate to `http://localhost:3000/meal-prep-hub.html`) rather than
guessing — this file has no test suite. `localStorage.clear()` before testing fresh
state since state persists across reloads.

## Architecture (single `<script>` block, no modules)

- `RECIPES` — array of 11 recipes. Each has `id, name, tag, emoji, color, veggie`
  (default VEGGIES id), `sauce` (macros), `ingredients` (string array, index 0 is
  always the chicken line and gets overwritten dynamically), `method` (string array),
  `tip`, `levers`.
- Lever schema: `{ label, qty, unit, grams, item, note, cal, protein, carbs, fat, fiber }`
  — `qty`/`unit` are the original kitchen measurement (tbsp, tsp, oz, cup, avocado,
  tortilla) for shopping; `grams` is the gram add-on for scale-weighing. Both are
  per-serving; scale by `state.servings` at render time.
- `RICE` / `VEGGIES` — option lists with macros. Veggie macros assume the recipe's
  default 1/2 cup serving; `grams` field is the weight of that serving.
- Rice macros are based on jasmine white rice cooked in water (no oil/salt).
- State: single mutable `state` object + `setState(updates)` which merges, persists to
  `localStorage` (key `mealPrepHubState`), and does a full `render()`. No virtual DOM —
  `render()` rebuilds `innerHTML` from template strings. Click handling is one
  delegated listener attached once at boot (`data-*` attributes dispatch by name).
- `formatOriginalQty(qty, unit, servings)` and `formatGrams(grams, servings)` —
  scaling helpers. `leverBatchText(lv, servings)` combines both for the "📦 batch
  quantity" callouts.

## Conventions established by user feedback

- **Units**: shopping-relevant quantities (Rice picker, Ingredients list base
  sauce items) keep their natural unit (cups, tbsp, lbs) since that's what's on
  the package. Batch/portioning quantities (Bulk Levers, Vegetable section, Meal
  Breakdown, Ingredients-for-this-batch) show **original unit + gram add-on**,
  e.g. `8 tbsp (128g) peanut butter`. Count-based items (Cuties, tortillas as
  whole units) are not converted to grams.
- No collapsible sections for Ingredients/Method — always visible, no dropdown.
- Servings/batch-size slider lives inside the recipe detail view, not the home grid.
- Veggie picker is per-recipe (each recipe has a recommended default + override),
  not a global control.
- Phases (Reverse Diet / Bulk stages) were replaced with manual numeric inputs
  (Daily Calories / Protein / Fiber) — don't reintroduce a phase system.

## Known gotcha

When building a breakdown row via spread, the override key must come *after* the
spread or it gets silently clobbered by the source object's own field of the same
name: `{ ...rice, label: "..." }` not `{ label: "...", ...rice }` (rice/veggie/lever
objects all have their own `label`/`qty` fields).
