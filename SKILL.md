---
name: mviz
description: A chart & report builder for AI. Use when the user asks to show, display, render, or visualize data as a chart (bar, line, pie, scatter, area, bubble, histogram, etc.), a table, KPI, big number, dashboard, report, sparkline, heatmap, sankey, funnel, calendar, waterfall, or any composed data visualization. Generates compact markdown specs that render either inline (via visualize:show_widget) or as standalone HTML files (via the mviz CLI).
---

mviz v1.7.0

# mviz

Generate clean, data-focused charts and dashboards from compact JSON specs or markdown. Maximizes data-ink ratio with minimal chartjunk, gridlines, and decorative elements. Uses a 16-column grid layout system.

## Setup

No installation required. Use `npx -y -q mviz` which auto-downloads from npm. The `-q` flag reduces npm output while still showing lint errors.

For faster repeated use, install globally: `npm install -g mviz`

## What This Skill Does

mviz is two things:

1. **A spec language** — markdown with fenced code blocks describing components (charts, KPIs, tables, notes, etc.). One markdown file (or a single JSON spec) describes a whole visualization.
2. **A set of renderers** that turn that spec into HTML for different display contexts. Today there are two:
   - **`visualize:show_widget`** (inline) — when you're in claude.ai with the visualization tool available, render the mviz spec as a `widget_code` HTML fragment using claude.ai's design system (Chart.js, `var(--color-*)` tokens). The widget appears inline in the conversation.
   - **mviz CLI** (standalone) — `mviz dashboard.md -o report.html` produces a complete HTML document with mviz's own design system (mdsinabox theme, ECharts). Suitable for downloads, iframe tiles, print/PDF.

The mviz markdown spec is canonical; the renderer is chosen based on where the visualization will appear. Same spec, two renderings. A third-party could build a different renderer (Chart.js-based CLI, Slack-themed renderer, etc.) without changing the spec.

## Picking a renderer

**Always compose the mviz markdown spec first**, even when rendering inline. The spec is the canonical artifact — emit it in a fenced code block in your response so the user can copy / reuse / pipe-to-CLI later. Then render it with whichever renderer fits the request.

Before rendering, decide which renderer fits — and **state your inference** so the user can correct you. Some signals:

**Inline (`visualize:show_widget`):**
- "show me X" / "what was X" / "how did X change" / "render"
- A small visualization (typically 1–4 components) that flows with the surrounding prose
- The user is exploring data, not preparing something to share
- `visualize:show_widget` is available in this conversation

**Standalone (mviz CLI):**
- "make a report" / "send me the file" / "export to PDF" / "save"
- Multi-section composed view (5+ components, dividers, page breaks, full-width tables)
- The user wants something to save, share, print, or embed in an external host (iframe tile, dashboard cell)
- `visualize:show_widget` is unavailable (no inline rendering surface)

Default to inline when ambiguous. State the inference explicitly:

> *"I'll render this inline since you asked to see it in chat. If you'd rather a downloadable file, I can run the mviz CLI on the same spec — just say the word."*

If the user later asks for the file version after an inline render, the spec is already in the conversation — pipe it through `mviz ... -o out.html` without re-composing.

## Visual Style (mdsinabox theme — for the CLI renderer)

- **Font**: Helvetica Neue, Arial (clean sans-serif)
- **Signature**: Orange accent line at top of dashboards
- **Palette**: Blue primary, orange secondary, semantic colors (green=positive, amber=warning, red=error)
- **Background**: Paper (`#f8f8f8` light) / Dark (`#231f20` dark)
- **Principles**: High data-ink ratio, no chartjunk, minimal gridlines, data speaks for itself

## Renderer: mviz CLI (standalone HTML)

The CLI renderer produces a complete HTML document styled in the mdsinabox theme above. Output is suitable for direct browser viewing, PDF export, iframe embedding, or sharing as a file.

### Single Chart (JSON)

```bash
echo '<json_spec>' | npx -y -q mviz -o chart.html
```

### Dashboard from Markdown

```bash
npx -y -q mviz dashboard.md -o dashboard.html
```

### Dashboard from Folder

```bash
npx -y -q mviz my-dashboard/ -o dashboard.html
```

### Iframe-embedded output (`--embed`)

When the rendered HTML will live inside another product's iframe (mdw-turbo Prism tiles, dashboard cells, etc.), pass `--embed` to strip page chrome (red accent bar, title row, theme toggle) and apply a battery of post-processes that lean the body: CSS pruning, transparent backgrounds so the host's card chrome shows through, ECharts script dropped when no charts are present, marked dropped when no `marked.parse(...)` caller, JS minify, etc.

```bash
npx -y -q mviz dashboard.md --embed -o tile.html
```

Embed mode emits a complete HTML document (DOCTYPE + html + head + body intact). It's not the right tool for `visualize:show_widget` — for that, use the other renderer (see below).

## 16-Column Grid System

Components are sized using `size=[cols,rows]` syntax:

````markdown
```big_value size=[4,2]
{"value": 1250000, "label": "Revenue", "format": "currency0m"}
```
```bar size=[8,6]
{"title": "Sales", "x": "month", "y": "sales", "file": "data/sales.json"}
```
````

- **16 columns** total width (both portrait and landscape)
- **Row height**: ~32px per row unit (approximate - charts have padding)
- **Page capacity**: Portrait [16c × 30r], Landscape [16c × 22r]
- Components on same line share the row
- Empty line = new row

**Height Guidelines:**
| Row Units | Approximate Height | Good For |
|-----------|-------------------|----------|
| 2 | ~64px | KPIs, single-line notes |
| 4 | ~128px | Small tables, text blocks |
| 5-6 | ~160-192px | Standard charts |
| 8+ | ~256px+ | Dense tables, detailed charts |

For charts with many categories (10+ bars, 10+ rows in dumbbell), increase row units to prevent compression.

### Side-by-Side Layout

**Critical:** To place components side-by-side, their code blocks must have NO blank lines between them:

````markdown
```bar size=[8,5]
{"title": "Chart A", ...}
```
```line size=[8,5]
{"title": "Chart B", ...}
```
````

This renders Chart A and Chart B on the same row. Adding a blank line between them would put them on separate rows.

### Headings and Section Breaks

| Syntax | Effect |
|--------|--------|
| `# H1` | Page title |
| `## H2` | Section title |
| `### H3` | Light inline header (subtle, smaller text) |
| `---` | Visual divider line |
| `===` | Page break for printing |
| `empty_space` | Invisible grid cell spacer (default 4 cols × 2 rows) |

**Heading Guidelines:**
- Set the page title with either the frontmatter `title:` *or* a leading `# H1`, not both.
- Use `## H2` for section titles within a page (most common).
- Use `### H3` for lightweight subheadings that don't interrupt flow.

**Section vs Page Breaks:**
- Use `---` to separate logical sections visually. Content flows naturally to the next page when needed.
- Use `===` only when you explicitly want to force a new page (e.g., separating chapters or major report sections for PDF output).
- Never use `===` by default. Only add page breaks when the user specifically requests them.

### Default Sizes

| Component | Default Size | Notes |
|-----------|-------------|-------|
| `big_value` | [4, 2] | Fits 4 per row |
| `delta` | [4, 2] | Fits 4 per row |
| `sparkline` | [4, 2] | Compact inline chart |
| `bar`, `line`, `area` | [8, 5] | Half width |
| `pie`, `scatter`, `bubble` | [8, 5] | Half width |
| `funnel`, `sankey`, `heatmap` | [8, 5] | Half width |
| `histogram`, `boxplot`, `waterfall` | [8, 5] | Half width |
| `combo` | [8, 5] | Half width |
| `dumbbell` | [12, 6] | 3/4 width |
| `table` | [16, 4] | Full width |
| `textarea` | [16, 4] | Full width |
| `calendar` | [16, 3] | Full width |
| `xmr` | [16, 6] | Full width, tall |
| `mermaid` | [8, 5] | Half width (use `ascii: true` for text art) |
| `alert`, `note`, `text` | [16, 1] | Full width, single row |
| `empty_space` | [4, 2] | Invisible spacer |

### Recommended Size Pairings

| Layout Goal | Components | Sizes |
|-------------|------------|-------|
| 4 KPIs in a row | 4× `big_value` | [4,2] each |
| 5 KPIs in a row | 4× `big_value` + 1 wider | [3,2] + [4,2] |
| KPI + context | `big_value` + `textarea` | [3,2] + [13,2] |
| KPI + chart | `big_value` + `bar` | [4,2] + [12,5] |

### Example: Dense KPI Row

````markdown
```big_value size=[3,2]
{"value": 1250000, "label": "Revenue", "format": "currency0m"}
```
```big_value size=[3,2]
{"value": 8450, "label": "Orders", "format": "num0k"}
```
```big_value size=[3,2]
{"value": 2400000000, "label": "Queries", "format": "num0b"}
```
```delta size=[3,2]
{"value": 0.15, "label": "MoM", "format": "pct0"}
```
```delta size=[4,2]
{"value": 0.08, "label": "vs Target", "format": "pct0"}
```
````

This creates a row with 5 KPIs (3+3+3+3+4 = 16 columns).

### Example: Two Charts Side by Side

````markdown
```bar size=[8,6] file=data/region-sales.json
```
```line size=[8,6] file=data/monthly-trend.json
```
````

## Supported Types

**Charts:** bar, line, area, pie, scatter, bubble, boxplot, histogram, waterfall, xmr, sankey, funnel, heatmap, calendar, sparkline, combo, dumbbell, mermaid

**UI Components:** big_value, delta, alert, note, text, textarea, empty_space, table

### Table Formatting

Tables support column-level and cell-level formatting:

**Column options:** `bold`, `italic`, `type` ("sparkline" or "heatmap")

```json
{
  "type": "table",
  "columns": [
    {"id": "product", "title": "Product", "bold": true},
    {"id": "category", "title": "Category", "italic": true},
    {"id": "sales", "title": "Sales", "fmt": "currency"},
    {"id": "margin", "title": "Margin", "type": "heatmap", "fmt": "pct"},
    {"id": "trend", "title": "Trend", "type": "sparkline", "sparkType": "line"}
  ],
  "data": [
    {"product": "Widget", "category": "Electronics", "sales": 125000, "margin": 0.85, "trend": [85, 92, 88, 95, 102, 125]}
  ]
}
```

**Cell-level overrides:** Use `{"value": "text", "bold": true}` to override column defaults.

**Heatmap:** Applies color gradient from low to high values. Text auto-switches to white on dark backgrounds.

**Sparkline types:** `line`, `bar`, `area`, `pct_bar` (progress bar), `dumbbell` (before/after comparison)

**Sort & filter:**

| Field | Values | Default | Effect |
|-------|--------|---------|--------|
| `sortable` | `true` / `false` | `true` | Click a column header to sort. Cycles asc → desc → off. Numeric columns sort by raw value. |
| `filter` | `true` / `false` | `false` | When true, render a "Filter by any value…" search input above the table. Rows filter live on case-insensitive contains match across all cells. |

Example: `{"type": "table", "filter": true, "columns": [...], "data": [...]}`

### Note Types

Notes support three severity levels via `noteType`:

| Type | Border Color | Use For |
|------|--------------|---------|
| `default` | Red | Important notices (default) |
| `warning` | Yellow | Cautions, preliminary data |
| `tip` | Green | Best practices, pro tips |

Notes also support an optional `label` for bold prefix text:

```json
{"type": "note", "label": "Pro Tip:", "content": "Use keyboard shortcuts for faster navigation.", "noteType": "tip"}
```

### Specialized Chart Examples

**big_value** - Hero metrics with large display:
```json
{"type": "big_value", "value": 1250000, "label": "Revenue", "format": "currency0m"}
```
- Optional `comparison` object: `{"value": 10300, "format": "currency", "label": "vs last month"}` shows change with arrow

**dumbbell** - Before/after comparisons with directional coloring:
```json
{
  "type": "dumbbell",
  "title": "ELO Changes",
  "category": "team",
  "start": "before",
  "end": "after",
  "startLabel": "Week 1",
  "endLabel": "Week 2",
  "higherIsBetter": true,
  "data": [
    {"team": "Chiefs", "before": 1650, "after": 1720},
    {"team": "Bills", "before": 1600, "after": 1550}
  ]
}
```
- Green = improvement, Red = decline, Grey = no change
- `higherIsBetter: false` for rankings (lower = better)
- Labels auto-abbreviate large numbers (7450 → "7k")

**delta** - Change metrics with directional coloring:
```json
{"type": "delta", "value": 0.15, "label": "MoM Growth", "format": "pct0"}
```
- Positive values show green with ▲, negative show red with ▼
- Optional `comparison` object: `{"value": 0.05, "label": "vs Target"}`

**area** - Filled line chart for cumulative/volume data:
```json
{
  "type": "area",
  "title": "Daily Active Users",
  "x": "date",
  "y": "users",
  "data": [{"date": "Mon", "users": 1200}, {"date": "Tue", "users": 1450}]
}
```

**combo** - Bar + line with dual Y-axis:
```json
{
  "type": "combo",
  "title": "Revenue vs Growth Rate",
  "x": "quarter",
  "y": ["revenue", "growth_rate"],
  "data": [
    {"quarter": "Q1", "revenue": 1000000, "growth_rate": 0.15},
    {"quarter": "Q2", "revenue": 1200000, "growth_rate": 0.20}
  ]
}
```
- First y-field renders as bars, second as line
- Dual Y-axes with independent scales

**heatmap** - 2D matrix visualization:
```json
{
  "type": "heatmap",
  "title": "Activity by Hour",
  "xCategories": ["Mon", "Tue", "Wed", "Thu", "Fri"],
  "yCategories": ["9am", "12pm", "3pm", "6pm"],
  "format": "num0",
  "data": [[0, 0, 85], [1, 0, 90], [2, 0, 72]]
}
```
- `format` option applies to cell labels (e.g., `num0k`, `currency0k`, `pct`)

**funnel** - Conversion or elimination flows:
```json
{
  "type": "funnel",
  "title": "Sales Pipeline",
  "format": "num0",
  "data": [
    {"stage": "Leads", "value": 1000},
    {"stage": "Qualified", "value": 600},
    {"stage": "Proposal", "value": 300},
    {"stage": "Closed", "value": 100}
  ]
}
```
- `format` option applies to labels/tooltips (e.g., `currency_auto`, `pct`, `num0`)

**waterfall** - Cumulative change visualization:
```json
{
  "type": "waterfall",
  "title": "Revenue Bridge",
  "x": "item",
  "y": "value",
  "data": [
    {"item": "Start", "value": 1000, "isTotal": true},
    {"item": "Growth", "value": 200},
    {"item": "Churn", "value": -50},
    {"item": "End", "value": 1150, "isTotal": true}
  ]
}
```

**bubble** - Scatter with size dimension. Supports `series` for color grouping and `showLabels` for persistent labels:
```json
{
  "type": "bubble",
  "title": "Market Analysis",
  "x": "growth",
  "y": "profit",
  "size": "revenue",
  "series": "region",
  "label": "company",
  "data": [
    {"growth": 5, "profit": 20, "revenue": 100, "region": "US", "company": "Acme"},
    {"growth": 10, "profit": 15, "revenue": 200, "region": "EU", "company": "Beta"}
  ]
}
```

**sankey** - Flow diagrams showing relationships:
```json
{
  "type": "sankey",
  "title": "Traffic Sources",
  "data": [
    {"source": "Organic", "target": "Landing", "value": 500},
    {"source": "Paid", "target": "Landing", "value": 300},
    {"source": "Landing", "target": "Signup", "value": 400}
  ]
}
```

**mermaid** - Diagrams from Mermaid syntax (flowcharts, sequence, state, class, ER). Use array for multi-line code:
```json
{
  "type": "mermaid",
  "title": "User Flow",
  "code": [
    "graph TD",
    "  A[Start] --> B{Decision}",
    "  B -->|Yes| C[Action]",
    "  B -->|No| D[End]"
  ]
}
```

**mermaid (ASCII)** - ASCII/Unicode text-based diagrams (set `ascii: true`):
```json
{
  "type": "mermaid",
  "title": "Process Flow",
  "code": ["graph LR", "  A[Input] --> B[Process] --> C[Output]"],
  "ascii": true
}
```

**Mermaid lint rules** (errors that will fail validation):
- No `<br/>` tags in labels (render as literal text, not line breaks)
- No quoted labels like `A["text"]` in flowcharts (quotes appear in output)

## Number Format Options

| Format | Example | Use For |
|--------|---------|---------|
| `auto` | 1.000m, 10.00k | **Smart auto-format (recommended)** |
| `currency_auto` | $1.000m, $10.00k | Smart auto-format with $ prefix |
| `currency0m` | $1.2m | Millions |
| `currency0b` | $1.2b | Billions |
| `currency0k` | $125k | Thousands |
| `currency` | $1,250,000 | Detailed amounts |
| `num0m` | 1.2m | Millions |
| `num0b` | 1.2b | Billions |
| `num0k` | 125k | Thousands |
| `num0` | 1,250,000 | Detailed counts |
| `pct` | 15.0% | Percentage with decimal |
| `pct0` | 15% | Percentage integer |
| `pct1` | 15.0% | Percentage with 1 decimal |
| `date` | 2026-02-24 | Date column — string values pass through; numeric epoch ms render as `YYYY-MM-DD`. Tables sort by epoch ms. |
| `duration` | 1m7s | Duration column — numeric seconds render as `1m7s` / `25m39s`; strings pass through. Tables sort by total seconds. |

**Percentage formats:** `pct`, `pct0`, `pct1` always multiply by 100 for scalar components (`big_value`, `delta`) — pass `0.15` for `15%`. Charts and tables are **series-aware**: they inspect the column/series and skip the multiply when any value exceeds 1, so `[15, 22, 18]` renders as `15.0%`, `22.0%`, `18.0%` (not `1500%+`) while `[0.15, 0.22, 0.18]` still renders as `15.0%`, `22.0%`, `18.0%`. Pass `[0.8, 1.2]` for a growth-rate series and it'll render as `80%`, `120%`. If `big_value` / `delta` get a pct value > 1, the linter warns "did you mean value/100?".

**Smart formatting (`auto`/`currency_auto`) is recommended.** The `format` option applies to both axis labels and data labels on bar charts. It automatically picks the right suffix (k, m, b) based on magnitude and always shows 4 significant digits. Negative values are wrapped in parentheses: `(1.000m)`.

When no format is specified, smart formatting is used by default.

### Auto-Detected Axis Formatting

Chart axes automatically detect the appropriate format based on field names:

| Field Pattern | Auto Format | Example |
|---------------|-------------|---------|
| revenue, sales, price, cost, profit, amount | `currency_auto` | $1.250m |
| pct, percent, rate, ratio | `pct` | 15.0% |
| All other numeric fields | `auto` | 1.250m |

Override with an explicit `format` field in the chart spec.

## Columnar Data Format

The chart generator auto-detects columnar query results. Instead of manually converting `columns`/`rows` to `data`, pass the result directly:

```json
{
  "type": "bar",
  "title": "Sales by Region",
  "x": "region",
  "y": "sales",
  "columns": ["region", "sales"],
  "rows": [["North", 45000], ["South", 32000], ["East", 28000]]
}
```

This is automatically converted internally. No manual JSON reconstruction needed.

## Axis Bounds (yMin/yMax)

For line, area, bar, and combo charts, control y-axis range with `yMin` and `yMax`:

```json
{
  "type": "line",
  "title": "Elo Rating Trend",
  "x": "date",
  "y": "elo",
  "yMin": 1400,
  "data": [{"date": "Oct", "elo": 1511}, {"date": "Jan", "elo": 1636}]
}
```

Use `yMin` when:
- Data doesn't start at 0 (ratings, stock prices, temperatures)
- You want to emphasize relative changes over absolute values

Use `yMax` when:
- Labels are being cut off at the top of the chart
- You need headroom above the highest data point

## Validation & Lint Rules

The CLI validates specs automatically using built-in lint rules. Use `--lint` flag for validation-only mode:

```bash
npx -y -q mviz --lint dashboard.md  # Validate without generating HTML
```

### Lint Rules

| Rule | Severity | Trigger |
|------|----------|---------|
| `required-fields` | warning | Missing required fields like `x`, `y`, or `data` |
| `unknown-field` | warning | Field not recognized for the chart type |
| `time-series-sorted` | error | Time series data not in chronological order |
| `sankey-wrong-keys` | error | Using `from`/`to` instead of `source`/`target` |
| `big-value-string` | error | Passing `"62.5%"` string instead of `0.625` number |
| `duplicate-x-values` | warning | Duplicate values on x-axis |
| `mermaid-no-br-tags` | error | `<br/>` tags in mermaid code (render as literal text) |
| `mermaid-no-quoted-labels` | error | Quoted labels like `A["text"]` in flowcharts |

**Errors** exit with code 1. **Warnings** log to stderr but don't fail.

### Common Fixes

**Time series error:** Sort your data by date before passing to the chart.

**Sankey wrong keys:** Use `source`, `target`, `value` in your data:
```json
{"source": "A", "target": "B", "value": 100}
```

**big_value string:** Pass numeric value with format option:
```json
{"type": "big_value", "value": 0.625, "format": "pct0", "label": "Rate"}
```

## Troubleshooting

### Warning Messages

The generator outputs helpful warnings to stderr when issues are detected:

| Warning | Cause | Solution |
|---------|-------|----------|
| `Invalid JSON in 'bar' block` | Malformed JSON syntax | Check JSON syntax, ensure proper quoting |
| `Unknown component type 'bars'` | Typo in chart type | Use suggested type (e.g., `bar` not `bars`) |
| `Cannot resolve 'file=...'` | File reference without base directory | Use file path argument or inline JSON |
| `Row exceeds 16 columns` | Too many components in one row | Reduce component widths or split into rows |

Warnings include context like content previews, similar type suggestions, and section/row info.

### Labels Cut Off at Chart Edges

If data labels on bar, line, or area charts are being cut off at the top:

1. Find the maximum value in your data
2. Set `yMax` to ~10-15% higher than that value

**Example:** If max value is 200, set `"yMax": 220`

```json
{
  "type": "bar",
  "title": "Sales",
  "x": "month",
  "y": "sales",
  "yMax": 250,
  "data": [{"month": "Jan", "sales": 180}, {"month": "Feb", "sales": 220}]
}
```

This provides headroom for the label text above the bars.

## Data Generation Best Practice

**Use SQL to generate data files** instead of manually authoring JSON. This reduces errors and ensures data accuracy:

```sql
-- Generate chart data file
COPY (
  SELECT month, SUM(sales) as sales, SUM(revenue) as revenue
  FROM orders
  GROUP BY month
  ORDER BY month
) TO 'data/monthly-sales.json' (FORMAT JSON, ARRAY true);
```

Then reference the generated file:

````markdown
```bar file=data/monthly-sales.json
{"title": "Monthly Sales", "x": "month", "y": "sales"}
```
````

This approach:
- Ensures data accuracy (no manual transcription errors)
- Keeps data in sync with source systems
- Reduces token usage (SQL is more compact than JSON arrays)
- Makes updates easy (re-run query to refresh)

## File References (JSON and CSV)

Reference external data files to save tokens and enable data/visualization separation:

### JSON Files
````markdown
```bar size=[8,6] file=data/sales.json
```
````

### CSV Files (DuckDB Workflow)

CSV files work great with DuckDB for data exploration:

```bash
# Export query results to CSV
duckdb -csv -c "SELECT quarter, revenue FROM sales" > data/quarterly.csv
```

````markdown
```bar file=data/quarterly.csv
{"title": "Quarterly Revenue", "x": "quarter", "y": "revenue"}
```
````

- **CSV provides data**, inline JSON provides chart options (title, x, y, format)
- **Auto-detection**: If no inline options, first column = x, second column = y
- **Type conversion**: Numeric strings auto-convert to int/float

### Benefits of File References

| Approach | Best For |
|----------|----------|
| Inline JSON | Small, static specs |
| JSON files | Reusable chart configs |
| CSV files | DuckDB workflows, frequently updated data |

## Dashboard Markdown Format

````markdown
---
theme: light
title: My Dashboard
---

# Page Title

## Section Name

```big_value size=[4,2]
{"value": 125000, "label": "Revenue", "format": "currency0k"}
```
```bar size=[12,6] file=data/sales.json
```
````

**Rules:**
- `# Title` sets the page title (first occurrence only)
- `## Section` creates a new section with divider (border, spacing)
- `### Header` creates a soft header within the current section (no divider)
- `---` creates a section break (untitled, visual divider only)
- `===` creates a page break (forces new page when printing to PDF)
- `size=[cols,rows]` controls layout (16-column grid)
- `size=auto` auto-calculates size from data
- `file=path` references external JSON
- Empty lines = new rows

## Theme Toggle

Dashboards include a theme toggle button (top right) that switches between light and dark modes. All charts dynamically update when the theme changes.

Set the default theme in frontmatter:

```yaml
---
title: My Dashboard
theme: dark
orientation: landscape
print: true
---
```

| Option | Description |
|--------|-------------|
| `title` | Dashboard title displayed at top |
| `theme` | `light` (default) or `dark` |
| `orientation` | `portrait` (default) or `landscape` for print layout |
| `print` | When `true`, requires explicit `size=[cols,rows]` on all components |
| `continuous` | When `true`, removes section breaks between `#` headers for flowing layout |
| `embed` | When `true`, strips page chrome (red accent bar, title row, theme toggle) so the output can be tucked inside another host's frame (iframe tile, dashboard cell). Also available as the `--embed` CLI flag. |

**Page capacity:** Portrait fits 30 row units, landscape fits 22 row units (Letter paper, 0.5" margins).

The theme toggle affects all charts globally - individual chart `theme` settings are ignored in favor of the global toggle.

## Renderer: `visualize:show_widget` (inline)

The show_widget renderer lives inside Claude. Claude reads the mviz spec, translates it to a `widget_code` HTML fragment using claude.ai's design system (Chart.js + `var(--color-*)` tokens), and passes it to `visualize:show_widget`. The widget appears inline in the conversation.

### Workflow

1. **Compose the mviz markdown spec** and emit it visibly in your response, in a fenced code block. The spec is the canonical artifact — even when the user only wants an inline render, write the spec down so it's available for later (copy / reuse / pipe-to-CLI). Then render below it.

   ```markdown
   ```mviz
   ---
   title: Q3 sales
   ---
   ```big_value
   {"value": 1250000, "label": "Q3 Revenue", "format": "currency_auto"}
   ```
   ...
   ```

2. **State your renderer inference**, one short sentence. Example: *"Rendering inline since you asked to see it in chat — say the word if you want the file too."*

3. **Translate the spec to `widget_code`** following the mapping tables below. Use claude.ai's design tokens (`var(--color-*)`), Chart.js for charts, metric cards for KPIs.

4. **Call `visualize:show_widget`** with the `widget_code`.

5. **Write the surrounding explanation as normal response text**, outside the tool call. The widget is the visual; your prose carries the meaning.

### When this renderer is the right pick

Trigger inline rendering when **all** of these hold:
- The user is in claude.ai web/desktop (not Claude Code, not the API).
- `visualize:show_widget` is in your available tools.
- The visualization is composed of mviz component types that have chat-native equivalents (see mapping below).
- The visualization is small enough to emit naturally as `widget_code` — model emit time dominates latency, so 6–8 components is comfortable, 15+ starts to feel slow.

If any of those fails — mviz-only chart types like sankey/heatmap, very large dashboards, no `show_widget` — render through the mviz CLI instead and offer the file as a follow-up.

### What the show_widget renderer is NOT

- **Not the mviz CLI.** Don't invoke `npx mviz` to produce HTML for `show_widget`. The CLI's mdsinabox-styled output doesn't match the chat host, and the subprocess + larger emit costs latency. The CLI is a *peer renderer* used for standalone files.
- **Not a paste-mviz-output-into-widget_code path.** mviz CLI emits a complete document with its own design system; that wouldn't render correctly inside the widget sandbox even if `show_widget` accepted documents (it doesn't — it requires fragments).

### The translation: mviz spec → claude.ai-native HTML

mviz markdown describes structure. Read the spec, then write a `widget_code` HTML fragment using claude.ai's design system.

**Component type mapping**

| mviz type | claude.ai-native rendering |
|---|---|
| `big_value` | Metric card: surface card with `--color-text-secondary` 13px label above, 24px / weight 500 number below |
| `delta` | Metric card variant with colored sign (`--color-success` / `--color-danger`) + arrow glyph |
| `bar` | Chart.js `type: 'bar'` |
| `line` | Chart.js `type: 'line'` |
| `area` | Chart.js `type: 'line'` with `fill: true` |
| `pie` | Chart.js `type: 'pie'`. If the mviz spec sets `donut: true`, render as Chart.js `type: 'doughnut'` instead (mviz exposes donut as a *flag* on the pie type, not a separate component). |
| `scatter` | Chart.js `type: 'scatter'` |
| `bubble` | Chart.js `type: 'bubble'` |
| `histogram` | Chart.js `type: 'bar'` over pre-bucketed data |
| `sparkline` | Inline `<svg>` (3–5 line-width strokes) — too small to justify Chart.js |
| `table` | HTML `<table>` with `border-collapse: collapse`, `0.5px solid var(--color-border-tertiary)`, header row in `--color-background-secondary` |
| `note`, `alert` | Bordered callout: `border-left: 3px solid var(--color-{info,warning,success,danger})`, padding 8px 12px, label + body text |
| `text`, `textarea` | Render the markdown content as prose **in your response text outside the tool call** — not inside the widget. The `visualize:read_me` design guide is explicit: text goes in the response, visuals go in the tool. |
| `empty_space` | An empty grid cell — produce `<div></div>` with the right `grid-column: span N` |
| `boxplot`, `waterfall`, `xmr`, `sankey`, `funnel`, `heatmap`, `calendar`, `combo`, `dumbbell`, `mermaid` | No clean Chart.js equivalent. If the visualization centers on one of these, fall back to a standalone `mviz` export and don't render inline. |

**Layout mapping**

mviz uses a 16-col grid with explicit `size=[cols, rows]` directives. Inside a 680px-wide `show_widget` container, that translates to:

```css
.mviz-grid {
  display: grid;
  grid-template-columns: repeat(16, 1fr);
  gap: 8px;
}
.mviz-grid > .item-12 { grid-column: span 12; }
.mviz-grid > .item-8  { grid-column: span 8; }
.mviz-grid > .item-4  { grid-column: span 4; }
```

For simple 2–3 component widgets you can also use claude.ai's recommended responsive pattern:

```css
display: grid;
grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
gap: 12px;
```

This auto-fits to the host's chosen width, which is the right choice when the exact column count is not load-bearing.

**Format hint mapping**

mviz format strings → JS formatters used inside Chart.js label/tooltip callbacks and metric-card text:

| mviz `fmt` | JS expression (assume `v` is the number) |
|---|---|
| `currency_auto`, `auto` | Use the helper below — the smart-format contract has magnitude tiers that don't fit on one line |
| `currency` | `'$' + v.toLocaleString('en-US')` (e.g. `1250000` → `$1,250,000`) |
| `currency0k` | `'$' + Math.round(v/1000) + 'k'` (e.g. `125000` → `$125k`) |
| `currency0m` | `'$' + (v/1e6).toFixed(1) + 'm'` (e.g. `1250000` → `$1.2m`) |
| `pct`, `pct1` | `(v * 100).toFixed(1) + '%'` — mviz multiplies pct by 100, so pass the decimal (`0.15` → `15.0%`). For chart series mviz auto-detects whether values are already in pct units; mirror that if the data is ambiguous. |
| `pct0` | `Math.round(v * 100) + '%'` (e.g. `0.15` → `15%`) |
| `num0` | `v.toLocaleString('en-US')` (e.g. `1250` → `1,250`) |
| `num1` | `v.toFixed(1)` (e.g. `1.234` → `1.2`) |
| `num0k` | `Math.round(v/1000) + 'k'` (e.g. `1250` → `1k`) |

The `currency_auto` / `auto` helper, mirroring mviz's `smartFormatNumber`. ~4 significant digits, magnitude tier with suffix, negatives in parentheses — match these exact thresholds so the inline render and the standalone report show identical numbers:

```js
function fmtAuto(v, sym = '') {
  const neg = v < 0, abs = Math.abs(v);
  let s;
  if (abs >= 1e9) {
    const x = abs / 1e9;
    s = sym + (x >= 100 ? x.toFixed(1) : x >= 10 ? x.toFixed(2) : x.toFixed(3)) + 'b';
  } else if (abs >= 1e6) {
    const x = abs / 1e6;
    s = sym + (x >= 100 ? x.toFixed(1) : x >= 10 ? x.toFixed(2) : x.toFixed(3)) + 'm';
  } else if (abs >= 1e4) {
    const x = abs / 1e3;
    s = sym + (x >= 100 ? x.toFixed(1) : x.toFixed(2)) + 'k';
  } else if (abs >= 1e3) {
    s = sym + abs.toLocaleString('en-US', { maximumFractionDigits: 0 });
  } else {
    s = sym + (abs === Math.floor(abs) ? abs : abs.toFixed(2));
  }
  return neg ? '(' + s + ')' : s;
}
// currency_auto: fmtAuto(v, '$')      → 1_250_000 → "$1.250m", 12_500 → "$12.50k", 1_250 → "$1,250"
// auto:          fmtAuto(v)            → 1_250_000 →  "1.250m", 12_500 →  "12.50k", 1_250 →  "1,250"
```

Always `Math.round` / `.toFixed(n)` displayed numbers — `0.1 + 0.2` is `0.30000000000000004` and that artifact will leak into the widget if you don't.

**Color mapping**

Chart.js renders to `<canvas>` and cannot resolve CSS variables. Use hardcoded hex from claude.ai's color ramps (the `c-{ramp}` stops listed in the `visualize:read_me` design module). A safe default palette for mviz series:

- Primary (mviz blue `#0777b3`) → `c-blue` 600 `#185FA5`
- Secondary (mviz orange `#bd4e35`) → `c-coral` 600 `#993C1D`
- Positive (mviz green `#2d7a00`) → `c-green` 600 `#3B6D11`
- Warning (mviz amber `#e18727`) → `c-amber` 600 `#854F0B`
- Error (mviz red `#bc1200`) → `c-red` 600 `#A32D2D`
- Tertiary / neutral → `c-gray` 600 `#5F5E5A`

For HTML elements (cards, table borders, text), use `var(--color-text-primary)` / `var(--color-background-secondary)` / `var(--color-border-tertiary)` and friends — they auto-adapt to dark mode.

### Chart.js setup (canonical patterns from `visualize:read_me`)

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
```

Wrap every canvas in a sized div:

```html
<div style="position: relative; width: 100%; height: 240px;">
  <canvas id="c1" role="img" aria-label="Sales by region — NA $540k, EU $380k, APAC $220k, LATAM $110k">
    Sales by region (bar chart). NA 540, EU 380, APAC 220, LATAM 110.
  </canvas>
</div>
```

Constraints to honor:

- `responsive: true, maintainAspectRatio: false` — height goes on the wrapper div only, never on the canvas itself.
- Horizontal bar charts: wrapper height ≥ `(bars * 40) + 80` px.
- ≤12 categories where every label must be visible: `scales.x.ticks: { autoSkip: false, maxRotation: 45 }`.
- Bubble/scatter: pad `scales.{x,y}.{min,max}` ~10 % beyond data range so radii don't clip.
- Negative currency: `-$5M` not `$-5M` — sign before symbol. Use a formatter `(v) => (v < 0 ? '-' : '') + '$' + Math.abs(v) + 'M'`.

Default Chart.js legends use round dots and no values — disable them and emit your own legend as inline HTML (small color squares, tight spacing, include the value/percentage when categorical):

```js
plugins: { legend: { display: false } }
```

```html
<div style="display: flex; flex-wrap: wrap; gap: 16px; margin-bottom: 8px;
            font-size: 12px; color: var(--color-text-secondary);">
  <span style="display: flex; align-items: center; gap: 4px;">
    <span style="width: 10px; height: 10px; border-radius: 2px; background: #185FA5;"></span>NA $540k
  </span>
  <!-- repeat -->
</div>
```

### Metric card (for `big_value` and `delta`)

```html
<div style="background: var(--color-background-secondary); border-radius: var(--border-radius-md);
            padding: 1rem;">
  <div style="font-size: 13px; color: var(--color-text-secondary);">Q3 Revenue</div>
  <div style="font-size: 24px; font-weight: 500; color: var(--color-text-primary);
              line-height: 1; margin-top: 4px;">$1.250m</div>
</div>
```

Use in grids of 2–4 with `gap: 12px`. For `delta`, add a small directional indicator with `color: var(--color-success)` or `var(--color-danger)`.

### Worked example

Spec (mviz markdown):

````markdown
---
title: Q3 sales
---

```big_value size=[4,2]
{"value": 1250000, "label": "Q3 Revenue", "format": "currency_auto"}
```
```big_value size=[4,2]
{"value": 0.184, "label": "Growth", "format": "pct1"}
```

```bar size=[16,5]
{"title": "Sales by region", "x": "region", "y": "sales", "format": "currency0k",
 "data": [{"region":"NA","sales":540000},{"region":"EU","sales":380000},
          {"region":"APAC","sales":220000},{"region":"LATAM","sales":110000}]}
```
````

Inline render (the `widget_code` you pass to `visualize:show_widget`):

```html
<h2 class="sr-only">Q3 sales overview: $1.250m revenue, 18.4% growth, regional breakdown.</h2>
<div style="display: grid; grid-template-columns: repeat(16, 1fr); gap: 12px;">
  <div style="grid-column: span 4; background: var(--color-background-secondary);
              border-radius: var(--border-radius-md); padding: 1rem;">
    <div style="font-size: 13px; color: var(--color-text-secondary);">Q3 Revenue</div>
    <div style="font-size: 24px; font-weight: 500; margin-top: 4px;">$1.250m</div>
  </div>
  <div style="grid-column: span 4; background: var(--color-background-secondary);
              border-radius: var(--border-radius-md); padding: 1rem;">
    <div style="font-size: 13px; color: var(--color-text-secondary);">Growth</div>
    <div style="font-size: 24px; font-weight: 500; margin-top: 4px;
                color: var(--color-success);">+18.4%</div>
  </div>
  <div style="grid-column: span 16;">
    <div style="font-size: 14px; font-weight: 500; margin-bottom: 8px;">Sales by region</div>
    <div style="position: relative; width: 100%; height: 240px;">
      <canvas id="mviz-bar-1" role="img"
              aria-label="Sales by region — NA $540k, EU $380k, APAC $220k, LATAM $110k">
        NA 540, EU 380, APAC 220, LATAM 110.
      </canvas>
    </div>
  </div>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<script>
  new Chart(document.getElementById('mviz-bar-1'), {
    type: 'bar',
    data: {
      labels: ['NA', 'EU', 'APAC', 'LATAM'],
      datasets: [{ label: 'Sales', data: [540000, 380000, 220000, 110000], backgroundColor: '#185FA5' }],
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        y: { ticks: { callback: (v) => '$' + Math.round(v/1000) + 'k' } },
        x: { ticks: { autoSkip: false, maxRotation: 45 } },
      },
    },
  });
</script>
```

### Constraints + things you must not do

- **Do not invoke the `mviz` CLI for `show_widget` output.** This path renders without it. The CLI is only for standalone HTML reports.
- **Do not paste mviz's own HTML into `show_widget`.** mviz emits its own styled fragments — they will not match the chat host and the bytes are wasted.
- No `<!DOCTYPE>`, `<html>`, `<head>`, or `<body>` in the widget — `show_widget` rejects documents.
- No `position: fixed`. It collapses the iframe's auto-sized viewport.
- Only load from the allowlist: `cdnjs.cloudflare.com`, `esm.sh`, `cdn.jsdelivr.net`, `unpkg.com`, `fonts.googleapis.com`, `fonts.gstatic.com`. Chart.js from `cdnjs.cloudflare.com` is the canonical source.
- Tables with more than ~3 columns or any prose-y content: render as markdown in your response text instead of inside the widget. The `visualize:read_me` design guide is explicit about this.

### Falling back to the CLI renderer

When the inline renderer isn't a fit, the spec is still useful — pipe it through the CLI:

- **Chart type with no Chart.js equivalent** (sankey, heatmap, calendar, etc.) — render via the CLI, offer the file. Mention that the mviz design system kicks in for the standalone output.
- **`visualize:show_widget` is not available** — render via the CLI and present as an artifact.
- **User is in Claude Code** — render via the CLI, write the file path, they'll open it in a browser. No inline widget surface in the TUI.
- **Visualization exceeds the inline size budget** (15+ components, dense tables with sparklines, multi-section reports) — render via the CLI. Offer inline as a follow-up if they want a smaller summary view.
- **Single tiny visual** (one sparkline, one 3-cell metric) — consider hand-rolling SVG directly via `show_widget` without going through the mviz spec at all. mviz earns its keep when composing multiple components; for a one-off icon it's overkill.

## Custom Themes

Load custom brand colors and fonts from a YAML file:

```bash
npx -y -q mviz --theme my_theme.yaml dashboard.md -o dashboard.html
```

Example theme file:
```yaml
name: brand-colors
extends: light

colors:
  primary: "#1a73e8"
  secondary: "#ea4335"

palette:
  - "#1a73e8"
  - "#ea4335"
  - "#fbbc04"

fonts:
  family: "'Roboto', sans-serif"
  import: "https://fonts.googleapis.com/css2?family=Roboto&display=swap"
```

Custom themes merge with defaults - only specify what you want to override.

## Print and PDF Support

Charts are optimized for printing to PDF:

- **High-Quality Rendering**: Uses SVG renderer for crisp vector graphics at any zoom level
- **No Page Breaks**: CSS prevents charts and tables from being split across pages
- **All Labels Visible**: Category labels always shown with 45° rotation to fit

When printing dashboards to PDF, all content stays intact without being cut off mid-chart.

## JSON Formatting for Editability

**Use formatted (multi-line) JSON** when data may need editing. This enables smaller, more precise edits:

````markdown
```bar size=[8,5]
{
  "title": "Monthly Sales",
  "x": "month",
  "y": "sales",
  "data": [
    {"month": "Jan", "sales": 120},
    {"month": "Feb", "sales": 150},
    {"month": "Mar", "sales": 180}
  ]
}
```
````

**Benefits:**
- Each data point on its own line enables targeted edits
- Changing one value: ~30 chars vs ~200+ chars with compact JSON
- Easier to review diffs in version control

**When to use compact JSON:**
- Very small specs (< 100 chars)
- Data that won't change
- Single-line values like `{"value": 1250000, "label": "Revenue"}`

## JSON Schema

mviz specs can be validated using the JSON Schema at:

```
https://raw.githubusercontent.com/matsonj/mviz/main/schema/mviz.schema.json
```

Add `$schema` to enable editor autocomplete and validation:

```json
{
  "$schema": "https://raw.githubusercontent.com/matsonj/mviz/main/schema/mviz.schema.json",
  "type": "bar",
  "title": "Sales",
  ...
}
```

## Color Palette (mdsinabox theme)

| Color | Hex | Use |
|-------|-----|-----|
| Primary Blue | `#0777b3` | Primary series |
| Secondary Orange | `#bd4e35` | Secondary series, accent |
| Info Blue | `#638CAD` | Tertiary, informational |
| Positive Green | `#2d7a00` | Success, positive values |
| Warning Amber | `#e18727` | Warnings |
| Error Red | `#bc1200` | Errors, negative emphasis |

See `reference/chart-types.md` for complete documentation.

## Your Role

You are an analytics assistant helping a human who has decision-making context that you lack. Your job is to present data clearly and surface patterns worth investigating—not to draw conclusions or make recommendations.

**Key principles:**
- Use a matter-of-fact tone. State what the data shows, not what it means.
- Design analysis that invites further questions, not analysis that closes them.
- Surface anomalies and patterns without assuming their cause or significance.
- Let the human add context and make decisions.

For additional guidance on creating effective data visualizations—including Tufte-inspired principles, anti-patterns to avoid, and layout examples—see `Best_practices.md`.

## Feedback

Having issues with mviz? Ask Claude to create a friction log documenting the problem, then open it as an issue at https://github.com/matsonj/mviz/issues
