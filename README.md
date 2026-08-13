# Sigma Decomposition Tree plugin

A configurable decomposition / KPI tree for Sigma. Each box shows a **main value**, a
**% variance** (▲/▼, green/red by sign), and **two bottom-corner metrics**. In **edit
mode** you build the tree by hand: add a root box, add any number of child boxes under
any box (arbitrary depth), and the layout + connecting arrows auto-reflow and scale to fit.

## Data model
- **Source:** any element — including the **raw table**. Each field you bind is **aggregated
  across the column** by the method you choose, so you don't need a pre-summarized element.
- **Per field (main value, variance, comparison, each corner) you pick a column + an
  aggregation:** Sum, Average, Min, Max, Count, Count distinct, First, or Last. Aggregation
  results are cached until the data changes, so redraws stay fast.
- **Only the main value is required** — variance and the two bottom corners are optional. A box
  can show just the big KPI number.
- **Structure + all box config** is stored as JSON in the workbook (the `config` editor field —
  managed by the plugin, no need to hand-edit it).

### Large tables & Sigma metrics
Sigma delivers the plugin a **capped window** of a very large element's rows, so a client-side
Sum/Avg over a raw multi-million-row table may reflect only part of the data. For accurate
totals on big tables, bind a **Sigma metric** (a measure aggregated server-side in the data
model) as the column and set its aggregation to **First** or **Average** — the metric already
carries the correct value. Use plugin-side Sum/Avg/etc. on grouped or reasonably-sized elements.

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
1. Add the plugin element to a page; in the editor panel set **Source** to your element
   (the main table is fine).
2. Turn on **Edit mode**. Click **＋ Add box**, then the **✎** on a box to pick a **column or
   metric** and an **aggregation** for the value / variance / corners. Any column you bind is
   **auto-registered** — the plugin adds it to "Columns used by the tree" for you so Sigma
   delivers its data. (You can also pre-populate that list manually in the editor panel.)
   Use the **＋** below any box to add children.
   - **Metrics / measures** appear in the pickers alongside dimensions. For a metric (already
     aggregated by Sigma), set the aggregation to **First** so the box shows its value as-is.
4. Turn **Edit mode** off for viewers. "Scale to fit" (toolbar) keeps the whole tree
   visible; turn it off to render at full size and scroll.

## Variance display
Each box has a **Variance mode**:
- **Bound column (direct)** — shows a variance column you already computed in Sigma.
- **Computed: % change vs comparison** — `(main value − comparison) ÷ comparison`, shown as a %.
- **Computed: delta vs comparison** — `main value − comparison`, shown in the value's format.

Arrow direction and color come from the sign, flipped when "Higher is better" is unchecked.

## Formatting
Each numeric field (value, variance, and both corners) has its own **Format** controls:
- **Type:** Auto (use the column's Sigma format), Number, Currency (`$`), Percent (`%`), or
  **Compact (K/M/B/T)** SI units.
- **Decimals:** fixed decimal places (blank = automatic).
- **Prefix / Suffix:** free text, e.g. prefix `$`, `€`; suffix `pp`, `bps`, ` units`.

Examples: a raw sum `25365929836087` → Compact + prefix `$` → **$25.4T**; a ratio column →
Percent, 1 decimal → **19.3%**. Negative values place the sign before the prefix (`-$0.02`).

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
