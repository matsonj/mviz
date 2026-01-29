---
name: mviz
description: A chart & report builder designed for use by AI.
---

mviz v1.5.0

# mviz

Generate clean, data-focused charts and dashboards from compact JSON specs or markdown. Maximizes data-ink ratio with minimal chartjunk, gridlines, and decorative elements. Uses a 16-column grid layout system.

## Setup

No installation required. Use `npx -y -q mviz` which auto-downloads from npm. The `-q` flag reduces npm output while still showing lint errors.

For faster repeated use, install globally: `npm install -g mviz`

## What This Skill Does

Converts minimal JSON specifications into standalone HTML visualizations using ECharts. Instead of writing 50-100 lines of chart code, write a compact spec that gets expanded into a full HTML artifact with professional styling.

## Visual Style (mdsinabox theme)

- **Font**: Helvetica Neue, Arial (clean sans-serif)
- **Signature**: Orange accent line at top of dashboards
- **Palette**: Blue primary, orange secondary, semantic colors (green=positive, amber=warning, red=error)
- **Background**: Paper (`#f8f8f8` light) / Dark (`#231f20` dark)
- **Principles**: High data-ink ratio, no chartjunk, minimal gridlines, data speaks for itself

## How to Use

### Single Chart (JSON)

```bash
echo '<json_spec>' | npx -y -q mviz > chart.html
```

### Dashboard from Markdown

```bash
npx -y -q mviz dashboard.md > dashboard.html
```

### Dashboard from Folder

```bash
npx -y -q mviz my-dashboard/ > dashboard.html
```

## 16-Column Grid System

Components are sized using `size=[cols,rows]` syntax:

````markdown
```big_value size=[4,2]
{"value": 1250000, "label": "Revenue", "format": "usd0m"}
```
```bar size=[8,6]
{"title": "Sales", "x": "month", "y": "sales", "file": "data/sales.json"}
```
````

- **16 columns** total width
- **Row height**: ~32px per row unit (approximate - charts have padding)
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
| `# H1` | Major section title |
| `## H2` | Section title |
| `### H3` | Light inline header (subtle, smaller text) |
| `---` | Visual divider line |
| `===` | Page break for printing |
| `===` | Explicit page break: forces new page in PDF |
| `empty_space` | Invisible grid cell spacer (default 4 cols × 2 rows) |

**Heading Guidelines:**
- Use `# H1` for major document sections that warrant their own page when printed
- Use `## H2` for content sections within a page (most common)
- Use `### H3` for lightweight subheadings that don't interrupt flow
- In `continuous: true` mode, H1 page breaks are suppressed

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
{"value": 1250000, "label": "Revenue", "format": "usd0m"}
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
    {"id": "sales", "title": "Sales", "fmt": "usd"},
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
{"type": "big_value", "value": 1250000, "label": "Revenue", "format": "usd0m"}
```
- Optional `comparison` object: `{"value": 10300, "format": "usd", "label": "vs last month"}` shows change with arrow

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
- `format` option applies to cell labels (e.g., `num0k`, `usd0k`, `pct`)

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
- `format` option applies to labels/tooltips (e.g., `usd_auto`, `pct`, `num0`)

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

**bubble** - Scatter plot with size dimension:
```json
{
  "type": "bubble",
  "title": "Market Analysis",
  "x": "growth",
  "y": "profit",
  "size": "revenue",
  "data": [
    {"growth": 5, "profit": 20, "revenue": 100},
    {"growth": 10, "profit": 15, "revenue": 200}
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
| `usd_auto` | $1.000m, $10.00k | Smart auto-format with $ prefix |
| `usd0m` | $1.2m | Millions |
| `usd0b` | $1.2b | Billions |
| `usd0k` | $125k | Thousands |
| `usd` | $1,250,000 | Detailed amounts |
| `num0m` | 1.2m | Millions |
| `num0b` | 1.2b | Billions |
| `num0k` | 125k | Thousands |
| `num0` | 1,250,000 | Detailed counts |
| `pct` | 15.0% | Percentage with decimal |
| `pct0` | 15% | Percentage integer |
| `pct1` | 15.0% | Percentage with 1 decimal |

**Important:** Percentage formats expect decimal values (0.25 = 25%), not whole numbers.

**Smart formatting (`auto`/`usd_auto`) is recommended.** The `format` option applies to both axis labels and data labels on bar charts. It automatically picks the right suffix (k, m, b) based on magnitude and always shows 4 significant digits. Negative values are wrapped in parentheses: `(1.000m)`.

When no format is specified, smart formatting is used by default.

### Auto-Detected Axis Formatting

Chart axes automatically detect the appropriate format based on field names:

| Field Pattern | Auto Format | Example |
|---------------|-------------|---------|
| revenue, sales, price, cost, profit, amount | `usd_auto` | $1.250m |
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
{"value": 125000, "label": "Revenue", "format": "usd0k"}
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
continuous: true
---
```

| Option | Description |
|--------|-------------|
| `title` | Dashboard title displayed at top |
| `theme` | `light` (default) or `dark` |
| `continuous` | When `true`, removes section breaks between `#` headers for flowing layout |

The theme toggle affects all charts globally - individual chart `theme` settings are ignored in favor of the global toggle.

## Custom Themes

Load custom brand colors and fonts from a YAML file:

```bash
npx -y -q mviz --theme my_theme.yaml dashboard.md > dashboard.html
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
