---
name: mviz
description: Generate clean reports from compact JSON specs
---

# mviz

Generate standalone HTML visualizations from JSON specs. Output as an HTML artifact that users can view, download, or print to PDF.

## JSON Schema

All specs validate against:
```
https://raw.githubusercontent.com/matsonj/mviz/main/schema/mviz.schema.json
```

Use the schema for field names and options. This skill covers how to use each type.

## When to Use Each Type

| Need | Use | Why |
|------|-----|-----|
| Show ranking + values + trends | `table` with sparklines | One component shows 6+ dimensions |
| Time series shape matters | `line` | Patterns, inflection points visible |
| Compare 8+ categories | `bar` | Too many rows for a table |
| Correlation between variables | `scatter` | Spatial relationship is the point |
| Single key metric | `big_value` | Hero number with optional comparison |
| Change/delta display | `delta` | Directional coloring (green/red) |
| Explanatory text | `textarea` | Markdown content, integrate prose |
| Callouts/warnings | `note` | Bordered box with label |

**Default to tables.** A table with sparkline columns replaces 3-4 separate charts.

## Single Chart

This is a simple exapmle of a chart defined in json.

```json
{
  "type": "bar",
  "title": "Sales by Region",
  "x": "region",
  "y": "sales",
  "data": [
    {"region": "North", "sales": 45000},
    {"region": "South", "sales": 32000}
  ]
}
```

Common fields: `type`, `title`, `data`, `x`, `y`, `format` (usd_auto, pct, num0, etc.)

## Report Layout (16-Column Grid)

Reports are **markdown files** with JSON specs inside fenced code blocks. The code block language (e.g., ` ```bar `) determines the component type. Add `size=[cols,rows]` after the type to control placement.

**Grid:** 16 columns (~50px each), rows are 32px tall. Size with `size=[cols,rows]`.

| Size | Dimensions | Use For |
|------|------------|---------|
| `[4,2]` | 200px × 64px | KPIs, deltas (4 per row) |
| `[8,5]` | 400px × 160px | bar, line, scatter, pie (2 per row) |
| `[12,6]` | 600px × 192px | dumbbell (3/4 width) |
| `[16,4]` | 800px × 128px | table, textarea (full width) |
| `[16,6]` | 800px × 192px | calendar, xmr (full width, tall) |
| `[16,10]` | 800px × 320px | heatmap (full width, extra tall) |
| `[16,1]` | 800px × 32px | note, alert (compact) |

**Markdown features supported:**
- These features will use full width (16 cols)
  - `# H1` = major section with page break (for PDF)
  - `## H2` = section title, visual divider, no page break
  - `### H3` = light inline header (subtle, smaller text)
  - YAML frontmatter for `title`, `theme` (light/dark), `continuous: true` (suppresses H1 page breaks)
  - `---` = untitled section divider, `===` = explicit page break

**Placing components:**
- Thoughtfully design the layout to fill all 16 columns of the grid. Try to match heights when using elements in the same row but using overides for default sizes
- A letter page (8.5×11") fits ~30 row units. Plan layouts accordingly.
- Code blocks with NO blank line between them → same row (side-by-side).
- Empty line between blocks → new row

### Example Report

````markdown
---
title: Revenue Analysis Report
theme: light
---

```textarea size=[16,3]
{"content": "## Summary\n\nQ3 revenue reached **$98M**, up 23% YoY."}
```

```bar size=[8,5]
{"title": "By Region", "x": "region", "y": "revenue", "data": [...]}
```
```line size=[8,5]
{"title": "Monthly Trend", "x": "month", "y": "revenue", "data": [...]}
```

```table size=[16,5]
{"columns": [{"id": "market"}, {"id": "revenue", "fmt": "usd_auto"}, {"id": "trend", "type": "sparkline"}], "data": [...]}
```
````

## Table Features

Tables support inline visualizations:

```json
{
  "type": "table",
  "columns": [
    {"id": "name", "title": "Product", "bold": true},
    {"id": "revenue", "fmt": "usd_auto"},
    {"id": "margin", "type": "heatmap", "fmt": "pct"},
    {"id": "trend", "type": "sparkline", "sparkType": "line"}
  ],
  "data": [
    {"name": "Widget", "revenue": 125000, "margin": 0.35, "trend": [10, 15, 12, 18, 22]}
  ]
}
```

Column `type`: `sparkline` (line/bar/area/pct_bar/dumbbell) or `heatmap` (color gradient).

## Best Practices

**Key points:**
- Use a matter-of-fact tone for exposition. 
- Design your analysis in a way that invites further questions.
- You are an analytics assistant for human who lacks decision making context, so don't assume decisions. 
- Think carefully about how to use to charts to enrich your human's understanding of the data. 

Only if pressed by your human to improve your design, channel Edward Tufte to build a beautiful, impactful analysis.