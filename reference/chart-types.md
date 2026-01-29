# Chart Types Reference

Examples and usage guidance for all chart types and components.

**Field reference:** For complete field definitions, see the [JSON Schema](https://raw.githubusercontent.com/matsonj/mviz/main/schema/mviz.schema.json).

---

## Charts

### bar

Vertical or horizontal bar chart, supports grouping and stacking.

**Simple:**
```json
{
  "type": "bar",
  "title": "Sales by Category",
  "x": "category",
  "y": "sales",
  "data": [
    {"category": "Electronics", "sales": 45000},
    {"category": "Clothing", "sales": 32000},
    {"category": "Home", "sales": 28000}
  ]
}
```

**Grouped** (y as array):
```json
{
  "type": "bar",
  "title": "Q1 vs Q2 Sales",
  "x": "category",
  "y": ["q1", "q2"],
  "data": [
    {"category": "Electronics", "q1": 45000, "q2": 52000},
    {"category": "Clothing", "q1": 32000, "q2": 38000}
  ]
}
```

**Stacked:**
```json
{
  "type": "bar",
  "title": "Sales Breakdown",
  "x": "month",
  "y": ["online", "retail"],
  "stacked": true,
  "data": [
    {"month": "Jan", "online": 30000, "retail": 25000},
    {"month": "Feb", "online": 35000, "retail": 28000}
  ]
}
```

---

### line

Line chart for trends, supports multi-series.

**Simple:**
```json
{
  "type": "line",
  "title": "Revenue Trend",
  "x": "month",
  "y": "revenue",
  "data": [
    {"month": "Jan", "revenue": 85000},
    {"month": "Feb", "revenue": 92000},
    {"month": "Mar", "revenue": 88000},
    {"month": "Apr", "revenue": 105000}
  ]
}
```

**Multi-series:**
```json
{
  "type": "line",
  "title": "Revenue vs Costs",
  "x": "month",
  "y": ["revenue", "costs"],
  "data": [
    {"month": "Jan", "revenue": 85000, "costs": 60000},
    {"month": "Feb", "revenue": 92000, "costs": 65000}
  ]
}
```

---

### area

Area chart, supports stacking.

```json
{
  "type": "area",
  "title": "Traffic Sources",
  "x": "week",
  "y": ["organic", "paid", "referral"],
  "stacked": true,
  "data": [
    {"week": "W1", "organic": 1200, "paid": 800, "referral": 400},
    {"week": "W2", "organic": 1400, "paid": 900, "referral": 450}
  ]
}
```

---

### pie

Pie or donut chart.

```json
{
  "type": "pie",
  "title": "Market Share",
  "donut": true,
  "data": [
    {"name": "Product A", "value": 45},
    {"name": "Product B", "value": 30},
    {"name": "Product C", "value": 15},
    {"name": "Other", "value": 10}
  ]
}
```

---

### scatter

Scatter plot for correlation analysis. Supports multiple series via `series` field.

**Simple:**
```json
{
  "type": "scatter",
  "title": "Price vs Sales",
  "x": "price",
  "y": "units_sold",
  "data": [
    {"price": 10, "units_sold": 500},
    {"price": 15, "units_sold": 420},
    {"price": 20, "units_sold": 350}
  ]
}
```

**Multi-series (grouped by category):**
```json
{
  "type": "scatter",
  "title": "Height vs Weight by Gender",
  "x": "height",
  "y": "weight",
  "series": "gender",
  "data": [
    {"height": 170, "weight": 70, "gender": "Male"},
    {"height": 165, "weight": 55, "gender": "Female"},
    {"height": 180, "weight": 80, "gender": "Male"},
    {"height": 160, "weight": 50, "gender": "Female"}
  ]
}
```

---

### bubble

Bubble chart (scatter with size dimension).

```json
{
  "type": "bubble",
  "title": "Market Analysis",
  "x": "market_size",
  "y": "growth_rate",
  "size": "revenue",
  "data": [
    {"market_size": 100, "growth_rate": 0.15, "revenue": 5000000},
    {"market_size": 80, "growth_rate": 0.25, "revenue": 3000000}
  ]
}
```

---

### boxplot

Box plot for statistical distribution. Data format: `[min, Q1, median, Q3, max]`.

```json
{
  "type": "boxplot",
  "title": "Response Time Distribution",
  "categories": ["API A", "API B", "API C"],
  "data": [
    [10, 25, 35, 50, 80],
    [15, 30, 45, 60, 95],
    [5, 20, 30, 40, 60]
  ]
}
```

---

### histogram

Histogram for distribution visualization.

```json
{
  "type": "histogram",
  "title": "Age Distribution",
  "bins": 8,
  "data": [22, 25, 28, 30, 32, 35, 38, 40, 42, 45, 48, 50, 55, 60]
}
```

---

### waterfall

Waterfall chart showing cumulative effect. Use `"type": "total"` for grounded bars, `null` value for auto-calculated totals.

**Colors:** Green = increases, Red = decreases, Blue = totals

```json
{
  "type": "waterfall",
  "title": "Revenue Breakdown",
  "data": [
    {"name": "Starting", "value": 1000, "type": "total"},
    {"name": "Product Sales", "value": 450},
    {"name": "Services", "value": 180},
    {"name": "Returns", "value": -120},
    {"name": "Discounts", "value": -95},
    {"name": "Ending", "value": null, "type": "total"}
  ]
}
```

---

### xmr

XmR (Individual-Moving Range) control chart for statistical process control.

**Output:** Two stacked charts - X Chart (values with control limits) and MR Chart (moving ranges).

**Nelson Rules:** By default, highlights points violating statistical control rules. Disable with `"nelson_rules": false`.

**Simple array:**
```json
{
  "type": "xmr",
  "title": "Daily Output",
  "data": [45.2, 47.1, 44.8, 46.3, 45.9, 48.2, 44.1, 46.8, 45.5, 47.3]
}
```

**With labels:**
```json
{
  "type": "xmr",
  "title": "Weekly Measurements",
  "data": [
    {"label": "Week 1", "value": 45.2},
    {"label": "Week 2", "value": 47.1},
    {"label": "Week 3", "value": 44.8}
  ]
}
```

---

### sankey

Sankey diagram for flow visualization.

```json
{
  "type": "sankey",
  "title": "Budget Allocation",
  "data": [
    {"source": "Revenue", "target": "Operations", "value": 500000},
    {"source": "Revenue", "target": "Marketing", "value": 200000},
    {"source": "Revenue", "target": "R&D", "value": 300000},
    {"source": "Operations", "target": "Salaries", "value": 350000},
    {"source": "Operations", "target": "Infrastructure", "value": 150000}
  ]
}
```

---

### funnel

Funnel chart for conversion stages.

```json
{
  "type": "funnel",
  "title": "Sales Funnel",
  "format": "usd_auto",
  "data": [
    {"name": "Visitors", "value": 10000},
    {"name": "Leads", "value": 5000},
    {"name": "Qualified", "value": 2500},
    {"name": "Proposals", "value": 1000},
    {"name": "Closed", "value": 500}
  ]
}
```

---

### heatmap

2D heatmap with color scale. Data format: `[x_index, y_index, value]`.

```json
{
  "type": "heatmap",
  "title": "Activity by Day/Hour",
  "xCategories": ["Mon", "Tue", "Wed", "Thu", "Fri"],
  "yCategories": ["9am", "12pm", "3pm", "6pm"],
  "data": [
    [0, 0, 10], [1, 0, 15], [2, 0, 20], [3, 0, 18], [4, 0, 12],
    [0, 1, 25], [1, 1, 30], [2, 1, 35], [3, 1, 28], [4, 1, 22]
  ]
}
```

---

### calendar

Calendar heatmap (GitHub-style).

```json
{
  "type": "calendar",
  "title": "Daily Activity",
  "year": 2024,
  "data": [
    {"date": "2024-01-15", "value": 5},
    {"date": "2024-01-16", "value": 12},
    {"date": "2024-01-17", "value": 8}
  ]
}
```

---

### sparkline

Compact inline chart for trends. Types: `line`, `bar`, `area`.

```json
{
  "type": "sparkline",
  "title": "Revenue Trend",
  "sparkType": "line",
  "data": [42, 48, 45, 52, 58, 55, 62, 68, 72, 78, 85, 92]
}
```

---

### combo

Combined bar and line chart with optional dual axis.

```json
{
  "type": "combo",
  "title": "Sales & Growth",
  "x": "quarter",
  "bar": ["sales"],
  "line": ["growth"],
  "dualAxis": true,
  "data": [
    {"quarter": "Q1", "sales": 120000, "growth": 0.12},
    {"quarter": "Q2", "sales": 150000, "growth": 0.25},
    {"quarter": "Q3", "sales": 140000, "growth": -0.07},
    {"quarter": "Q4", "sales": 180000, "growth": 0.29}
  ]
}
```

---

### dumbbell

Before/after comparison with directional color-coding.

- **Green** = improvement, **Red** = decline, **Gray** = no change
- Set `higherIsBetter: false` for rankings (lower is better)

**Revenue growth:**
```json
{
  "type": "dumbbell",
  "title": "Revenue by Region",
  "category": "region",
  "start": "before",
  "end": "after",
  "startLabel": "2023",
  "endLabel": "2024",
  "data": [
    {"region": "East", "before": 30, "after": 45},
    {"region": "West", "before": 25, "after": 60},
    {"region": "North", "before": 40, "after": 35}
  ]
}
```

**Rankings (lower is better):**
```json
{
  "type": "dumbbell",
  "title": "League Rankings",
  "category": "team",
  "start": "week1",
  "end": "week10",
  "higherIsBetter": false,
  "data": [
    {"team": "Team Alpha", "week1": 5, "week10": 1},
    {"team": "Team Beta", "week1": 10, "week10": 3}
  ]
}
```

---

### mermaid

Renders Mermaid diagram syntax to SVG or ASCII. Supports flowcharts, sequence diagrams, state diagrams, class diagrams, and ER diagrams.

The `code` field accepts a string or array of strings (joined with newlines). Array format is easier to read and edit:

**Flowchart (array format - recommended):**
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

**Sequence diagram:**
```json
{
  "type": "mermaid",
  "title": "API Call",
  "code": [
    "sequenceDiagram",
    "  Client->>Server: Request",
    "  Server-->>Client: Response"
  ]
}
```

**ER diagram:**
```json
{
  "type": "mermaid",
  "title": "Data Model",
  "code": [
    "erDiagram",
    "  CUSTOMER ||--o{ ORDER : places",
    "  ORDER ||--|{ LINE_ITEM : contains",
    "  PRODUCT ||--o{ LINE_ITEM : includes"
  ]
}
```

**ASCII rendering** (set `ascii: true` for text-based output):
```json
{
  "type": "mermaid",
  "title": "Process Flow",
  "code": ["graph LR", "  A[Input] --> B[Process] --> C[Output]"],
  "ascii": true
}
```

**Pure ASCII** (no Unicode box-drawing characters, add `useAscii: true`):
```json
{
  "type": "mermaid",
  "title": "Simple Flow",
  "code": ["graph LR", "  A --> B --> C"],
  "ascii": true,
  "useAscii": true
}
```

**Lint rules** (these cause validation errors):
- No `<br/>` tags in labels - they render as literal text
- No quoted labels like `A["text"]` in flowcharts - quotes appear in output
```

---

## UI Components

### big_value

Large metric display for KPIs. Optional `comparison` shows change indicator.

**Basic (label below number):**
```json
{
  "type": "big_value",
  "value": 1250000,
  "label": "Total Revenue",
  "format": "usd"
}
```

**With title (you can use either `title` or `label` - both work the same when only one is provided):**
```json
{
  "type": "big_value",
  "value": 29.6,
  "title": "Points Per Game",
  "format": "num1"
}
```

**With header AND label (title becomes H2 header above, label goes below):**
```json
{
  "type": "big_value",
  "title": "Q4 Results",
  "value": 1250000,
  "label": "Total Revenue",
  "format": "usd"
}
```

**With comparison:**
```json
{
  "type": "big_value",
  "value": 1250000,
  "label": "Total Revenue",
  "format": "usd",
  "comparison": {
    "value": 0.15,
    "label": "vs Last Month",
    "format": "pct"
  }
}
```

---

### delta

Change indicator with arrow. Green for positive (or red if `positiveIsGood: false`).

**Basic (label below value):**
```json
{
  "type": "delta",
  "value": 0.15,
  "label": "vs Last Month",
  "format": "pct"
}
```

**With title (you can use either `title` or `label` - both work the same when only one is provided):**
```json
{
  "type": "delta",
  "value": -0.0041,
  "title": "Margin Gap",
  "format": "pct"
}
```

**With header AND label (title becomes header above, label goes below):**
```json
{
  "type": "delta",
  "title": "Q4 Impact",
  "value": -82769,
  "label": "Late vs On Time",
  "format": "usd_auto"
}
```

**Inverted (lower is better):**
```json
{
  "type": "delta",
  "value": -25,
  "label": "Bugs Fixed",
  "format": "num0",
  "positiveIsGood": false
}
```

---

### alert

Colored notification banner. Types: `info`, `success`, `warning`, `error`. Supports `**bold**` and `*italic*`.

```json
{
  "type": "alert",
  "message": "Q2 targets **exceeded** by 15%!",
  "alertType": "success"
}
```

---

### note

Information callout box. Types: `default` (red), `warning` (yellow), `tip` (green).

```json
{
  "type": "note",
  "label": "Pro Tip",
  "content": "Use **keyboard shortcuts** to navigate faster.",
  "noteType": "tip"
}
```

---

### text

Styled paragraph. Supports `**bold**` and `*italic*`.

```json
{
  "type": "text",
  "content": "This dashboard shows **key performance metrics** for Q2 2024."
}
```

---

### textarea

Markdown-formatted text area. Supports headers, lists, code blocks, blockquotes, links.

```json
{
  "type": "textarea",
  "title": "Summary",
  "content": "## Overview\n\nThis dashboard shows **key metrics** for Q2.\n\n- Revenue up 15%\n- Customer growth steady"
}
```

---

### empty_space

Invisible grid spacer.

```json
{"type": "empty_space"}
```

---

### table

Data table with formatting. Supports inline sparklines and heatmap columns.

**Basic:**
```json
{
  "type": "table",
  "title": "Sales Data",
  "columns": [
    {"id": "product", "title": "Product"},
    {"id": "sales", "title": "Sales", "align": "right", "fmt": "usd"},
    {"id": "margin", "title": "Margin", "align": "right", "fmt": "pct"}
  ],
  "data": [
    {"product": "Widget Pro", "sales": 125000, "margin": 0.35},
    {"product": "Gadget X", "sales": 98000, "margin": 0.28}
  ]
}
```

**With sparklines:**
```json
{
  "type": "table",
  "columns": [
    {"id": "product", "title": "Product"},
    {"id": "sales", "title": "Sales", "fmt": "usd0k"},
    {"id": "trend", "title": "Trend", "type": "sparkline", "sparkType": "line"}
  ],
  "data": [
    {"product": "Widget", "sales": 125000, "trend": [85, 92, 98, 108, 115, 125]}
  ]
}
```

**Column types:**
- `"type": "sparkline"` with `sparkType`: `line`, `bar`, `area`, `pct_bar`, `dumbbell`
- `"type": "heatmap"` - color gradient from low to high values

**Dumbbell sparklines:** Data as `[start, end]` array. Set `higherIsBetter: false` for metrics where lower is better.

```json
{"id": "change", "type": "sparkline", "sparkType": "dumbbell", "higherIsBetter": true}
```

**Cell overrides:** Use `{"value": "text", "bold": true}` to override column styling per cell.

---

## Format Options

| Format | Example | Description |
|--------|---------|-------------|
| `auto` | 1.000m | Smart auto-format (default) |
| `usd_auto` | $1.000m | Smart auto-format with $ |
| `usd` | $1,250,000 | Full dollars |
| `usd0k` | $125k | Compact thousands |
| `usd0m` | $1.2m | Compact millions |
| `pct` | 15.0% | Percentage with decimal |
| `pct0` | 15% | Percentage integer |
| `num0` | 1,250,000 | Number with commas |
| `num0k` | 125k | Compact thousands |

**Auto-detection:** Fields named `revenue`, `sales`, `price`, `cost` → `usd_auto`. Fields with `pct`, `percent`, `rate` → `pct`.

---

## Grid System

16-column grid. Use `size=[cols,rows]` in code block headers.

**Default sizes:**
| Component | Size | | Component | Size |
|-----------|------|-|-----------|------|
| `big_value`, `delta` | [4,2] | | `bar`, `line`, `area` | [8,5] |
| `table`, `textarea` | [16,4] | | `dumbbell` | [12,6] |
| `mermaid` | [8,5] | | | |
| `alert`, `note` | [16,1] | | `empty_space` | [4,2] |

**Auto-sizing:** Use `size=auto` to calculate from data.

---

## Dashboard Markdown

```markdown
---
title: My Dashboard
theme: light
continuous: true
---

# Title

## Section

```big_value size=[4,2]
{"value": 125000, "label": "Revenue", "format": "usd0k"}
```
```bar size=[12,6] file=data/sales.json
```
```

**Frontmatter:** `title`, `theme` (light/dark), `continuous` (removes section breaks)

**Breaks:** `---` = section break, `===` = page break (PDF)

**Side-by-side:** Code blocks with no blank line between them share the same row.

---

## File References

**JSON:** `file=data/sales.json` - complete spec in file

**CSV:** `file=data.csv` with inline options - data from CSV, config inline

```bash
duckdb -csv -c "SELECT quarter, revenue FROM sales" > data/quarterly.csv
```

```markdown
```bar file=data/quarterly.csv
{"title": "Quarterly Revenue", "x": "quarter", "y": "revenue"}
```
```

---

## Color Palette

| Color | Hex | Use |
|-------|-----|-----|
| Primary Blue | `#0777b3` | Primary series |
| Secondary Orange | `#bd4e35` | Secondary/accent |
| Positive Green | `#2d7a00` | Success |
| Warning Amber | `#e18727` | Warnings |
| Error Red | `#bc1200` | Errors |
