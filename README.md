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
- `index.html.bak` — the build immediately before the 2026-08-21 robustness patch.

## 2026-08-21 patch — needs deploying

Three changes, all backward compatible with existing workbooks:

1. **The layout JSON is split over four config entries** (`config`…`config4`, 4,000
   chars each) and re-joined on read. A single Sigma config text entry stops being
   delivered somewhere past a few KB, and the symptom is the worst kind: the element
   sits on "loading" forever with no error in the console, in the element, or in the
   spec. Reading still accepts the old single-entry form, so nothing already saved
   breaks. Writing also strips every field that matches the node default first, so
   editing a box in the UI no longer inflates the document.
2. **It can no longer fail silently.** The whole render path is wrapped, and any
   exception is drawn into the element along with a plumbing line — config chars,
   whether a source is set, how many columns were declared, how many arrived with
   data. The "waiting for columns" state shows the same line. If this plugin ever
   goes blank again, the element itself will say why.
3. **The resize observer is guarded.** `draw()` writes `stage.innerHTML`, which can
   change the stage's own client size (a scrollbar appearing or going away), so the
   observer could re-enter `draw()` indefinitely and peg the iframe. It now ignores
   sub-2px churn and coalesces to one animation frame.

After deploying, set `TREE_CHUNK=1` when running
`customer-value-dashboard/build.py` so the workbook writes the chunked form.
Until then leave it off — the currently hosted build reads `config` alone and would
see a truncated document.

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
