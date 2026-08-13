# Sigma Decomposition Tree plugin

A configurable decomposition / KPI tree for Sigma. Each box shows a **main value**, a
**% variance** (▲/▼, green/red by sign), and **two bottom-corner metrics**. In **edit
mode** you build the tree by hand: add a root box, add any number of child boxes under
any box (arbitrary depth), and the layout + connecting arrows auto-reflow and scale to fit.

## Data model
- **Source:** a single (aggregated) row element — each column is one KPI value.
  (If the element has multiple rows, the first non-null value of each column is used.)
- **Per box you pick:** main-value column, variance column (+ optional suffix like `pp`),
  "higher is better" direction, and a column + label for each bottom corner.
- **Structure + all box config** is stored as JSON in the workbook (the `config` editor field —
  managed by the plugin, no need to hand-edit it).

## Files
- `index.html` — the entire plugin (vanilla JS + CDN SDK, no build step).

## Local development
Serve the folder and register the local URL in Sigma:
```bash
npx serve sigma-decomposition-tree -l 3000
# or:  python3 -m http.server 3000 --directory sigma-decomposition-tree
```
Then in Sigma → **Administration → Plugins → Add Plugin**, use `http://localhost:3000`.

## Production hosting
Any static host works (Netlify, Vercel, S3+CloudFront, GitHub Pages). Host `index.html`
at a public URL and register that URL in Sigma.

## Using it in a workbook
1. Add the plugin element to a page; in the editor panel set **Source** to your
   single-row KPI element.
2. Turn on **Edit mode**. Click **＋ Add box**, then the **✎** on a box to choose its
   value / variance / corner columns. Use the **＋** below any box to add children.
3. Turn **Edit mode** off for viewers. "Scale to fit" (toolbar) keeps the whole tree
   visible; turn it off to render at full size and scroll.

## Variance display
Each box has a **Variance mode**:
- **Bound column (direct)** — shows a variance column you already computed in Sigma.
- **Computed: % change vs comparison** — `(main value − comparison) ÷ comparison`, shown as a %.
- **Computed: delta vs comparison** — `main value − comparison`, shown in the value's format.

For the direct mode, the column's own Sigma number format is respected (currency → `$0.07`,
percent → `19.3%`, etc.). Arrow direction and color come from the sign, flipped when
"Higher is better" is unchecked. Use the **Suffix** field for units Sigma can't format on
its own (e.g. `pp`).

## Click-to-filter + breadcrumb (optional)
In the editor panel, bind **Select → variable** (a text control variable) and/or
**Select → action** (a workbook action). In viewer mode, clicking a box writes that box's
title — or its optional **"Value sent on click"** override — to the variable and fires the
action, so the rest of the dashboard can filter/drill on the clicked KPI. In edit mode,
clicking anywhere on a box opens its config instead.

When a selection is active, a **breadcrumb bar** appears showing the path from the root to
the selected box. Click any ancestor crumb to re-filter to it, click the selected box again,
or hit **✕ Clear** to reset (writes an empty value to the variable and fires the action).
The selected box is outlined in blue.

## Collapse / expand
Any box with children shows a **▾ / ▸** toggle beneath it. Collapsing hides the subtree and
the tree reflows; a collapsed box shows a small child count (e.g. `▸ 2`). In **edit mode** the
collapsed state is saved with the workbook (a default for viewers); in **viewer mode** a
viewer's collapse/expand is session-only and doesn't change the saved layout.
