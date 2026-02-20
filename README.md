# mviz

Generate beautiful static reports for ad hoc analysis. A Claude skill that turns compact JSON specs into professional HTML visualizations.

![Light Mode Dashboard](LightMode.png)

![Dark Mode Dashboard](DarkMode.png)

## Why mviz?

**The highest-value analysis in any company is point-in-time, highly contextual, and not reused once the decision is made.**

Traditional BI tools optimize for reusability instead of usefulness. Useful analysis, the kind that drives critical decisions, needs something more:

- **Fast iteration**: Query data → visualize → refine → share
- **AI-native workflow**: Works seamlessly with Claude for data exploration
- **Static output**: Beautiful HTML/PDF reports, no infrastructure required
- **Minimal tokens**: Compact specs instead of verbose chart code

Instead of writing 50-100 lines of chart boilerplate, write a compact JSON spec that gets expanded into a full HTML artifact with ECharts.

## Quick Start

### 1. Connect Your Database

Connect Claude to your data using an MCP server:

- **[MotherDuck MCP](https://motherduck.com/docs/sql-reference/mcp/)** - Cloud data warehouse with DuckDB compatibility
- **[Local DuckDB MCP](https://github.com/motherduckdb/mcp-server-motherduck)** - Query local `.duckdb` files, Parquet, CSV, or S3 data

If you do not have a database available, you can also load CSV files directly, although the amount of data you can fit in context can be quite limiting.

### 2. Add the Skill

**Claude Web or Desktop:** Download [`mviz.skill`](https://gist.github.com/matsonj/1c13b656bca20b1d2cc6260309d8eb40) and add it to your project knowledge.

**Claude Code:** Run `npx skills add matsonj/mviz` or clone this repo and work from the directory.

### 3. Effective Use Tips

The best analysis follows four steps:

1. **Build context** — Get the data right. Query, filter, and explore until you understand what you're looking at.
2. **Develop narrative** — What's the story? What question are you answering? What pattern matters?
3. **First pass on viz** — Create an initial visualization. Don't overthink it.
4. **Refine based on what doesn't work** — Iterate. Change chart types, adjust formatting, add context.

Start by exploring your data with natural questions. Claude writes SQL queries behind the scenes and brings the results into context:

> *"Show me revenue by region for Q4"*

> *"What are our top 10 customers by lifetime value?"*

> *"Are there any anomalies in last month's sales data?"*

Once you've built up context and are ready to visualize, tell Claude to **"use mviz to report on this analysis"**. Claude generates a polished HTML report from the data you've explored.

### 4. Iterate

Refine your analysis by asking follow-up questions:

> *"Change that bar chart to a line chart"*

> *"Drill into the APAC region—what's driving that spike?"*

> *"Add a table showing the top 5 products by growth rate"*

### mviz Specific Guidance

mviz uses a 16-column grid.

> *"Make the bar chart wider"*

> *"Show two charts side by side at size=[8,6] each"*

> *"Make the KPIs smaller: size=[3,2] so 5 fit in a row"*

By default, it will use `size=auto` to let mviz calculate appropriate dimensions based on your data.

> [!TIP]
> There are more chart types available in the library than are included in the skill.md. You can tell Claude to look at the TypeScript source for more chart types if you really need them.

Each iteration builds on your existing context. When you're done, save the HTML or print to PDF.

## Supported Chart Types

| Type | Description | mviz.skill |
|------|-------------|:----------:|
| `bar` | Vertical/horizontal, grouped, stacked | ✓ |
| `line` | Single or multi-series with linear interpolation | ✓ |
| `area` | Simple or stacked area charts | |
| `pie` | Pie or donut charts | |
| `scatter` | 2D scatter plots | ✓ |
| `bubble` | Scatter with size dimension (auto-detects categorical axes) | |
| `boxplot` | Statistical box plots | |
| `histogram` | Distribution visualization | |
| `sankey` | Flow diagrams | |
| `funnel` | Conversion funnels | |
| `heatmap` | 2D color matrices | |
| `calendar` | GitHub-style calendar heatmaps | |
| `sparkline` | Compact inline charts | |
| `combo` | Combined bar + line with dual axes | |
| `waterfall` | Cumulative effect charts | |
| `xmr` | Statistical control charts (supports `yMin`/`yMax`) | |
| `dumbbell` | Before/after comparisons with directional color-coding | |

## UI Components

| Type | Description | mviz.skill |
|------|-------------|:----------:|
| `big_value` | Large KPI metric display | |
| `delta` | Change indicator with arrow | |
| `table` | Data tables with formatting and inline sparklines | ✓ |
| `alert` | Colored notification banners | |
| `note` | Information callout boxes | ✓ |
| `text` | Styled paragraphs | |
| `textarea` | Markdown-rendered text blocks | ✓ |
| `empty_space` | Layout spacing component | ✓ |

## File References (JSON and CSV)

Reference external files instead of embedding large JSON specs:

````markdown
```bar file=data/monthly-sales.json
```
````

### DuckDB Workflow

CSV files work great for data exploration with DuckDB:

```bash
# Export query results
duckdb -csv -c "SELECT month, revenue FROM sales GROUP BY 1" > data/monthly.csv
```

````markdown
```bar file=data/monthly.csv
{"title": "Monthly Revenue", "x": "month", "y": "revenue"}
```
````

CSV provides data, inline JSON provides chart options. Auto-detects x/y from first two columns if no options given.

## Report Markdown Format

````markdown
---
theme: light
title: My Report
---

# Page Title

## Section Name

```big_value size=[4,2]
{"value": 125000, "label": "Revenue", "format": "usd0m"}
```
```delta size=[4,2]
{"value": 0.15, "label": "vs Last Month", "format": "pct0"}
```

```bar size=[8,6] file=data/sales.json
```
```line size=[8,6] file=data/trend.json
```
````

### Layout Rules

- `# Title` creates a new section (first one also sets page title)
- `## Section` creates a subsection title (no visual divider)
- `---` creates a visual section divider
- `===` creates a page break for printing
- `size=[cols,rows]` controls 16-column grid layout
- `size=auto` auto-calculates size based on data
- `file=path` references external JSON
- Multiple blocks on same line = same row
- Empty lines = new rows

## 16-Column Grid System

| Component | Default Size | Notes |
|-----------|-------------|-------|
| `big_value`, `delta`, `sparkline` | [4, 2] | Fits 4 per row |
| `bar`, `line`, `area`, `pie` | [8, 5] | Half width |
| `scatter`, `bubble`, `combo`, `funnel` | [8, 5] | Half width |
| `dumbbell` | [12, 6] | 3/4 width for comparisons |
| `table`, `heatmap` | [16, 4-10] | Full width |
| `xmr`, `calendar` | [16, 6] | Full width, tall |

## Table with Sparklines

Tables support inline sparkline columns for trend visualization:

```json
{
  "type": "table",
  "columns": [
    {"id": "product", "title": "Product"},
    {"id": "sales", "title": "Sales", "fmt": "usd"},
    {"id": "trend", "title": "Trend", "type": "sparkline", "sparkType": "line"},
    {"id": "progress", "title": "Goal", "type": "sparkline", "sparkType": "pct_bar", "width": 100}
  ],
  "data": [
    {"product": "Widget", "sales": 125000, "trend": [85, 92, 88, 95, 102, 110, 125], "progress": 0.85}
  ]
}
```

Sparkline types: `line`, `bar`, `area`, `pct_bar` (progress bar), `dumbbell` (before/after)

## Format Options

| Format | Output | Description |
|--------|--------|-------------|
| `auto` | 1.000m, 10.00k | **Smart auto-format (default)** |
| `usd_auto` | $1.000m, $10.00k | Smart auto-format with $ |
| `usd0m` | $1.2m | Millions |
| `usd0k` | $125k | Compact thousands |
| `usd` | $1,250,000 | Full dollars |
| `pct0` | 15% | Percentage integer |
| `pct` | 15.0% | Percentage with decimal |
| `pct1` | 15.0% | Percentage with 1 decimal |
| `num0` | 1,250 | Number with commas |

Smart formatting automatically picks the right suffix (k, m, b) based on magnitude and shows 4 significant digits. Negative values display in parentheses: `(1.000m)`.

### Auto-Detected Formatting

Chart axes automatically detect the appropriate format based on field names:

| Field Pattern | Auto Format | Example |
|---------------|-------------|---------|
| revenue, sales, price, cost, profit | `usd_auto` | $1.250m |
| pct, percent, rate, ratio | `pct` or `pct0` | 15.0% |
| All other fields | `auto` | 1.250m |

Override with an explicit `format` field in the chart spec.

## Theme Toggle

Reports include a theme toggle button (top right) that switches between light and dark modes. All charts dynamically update when the theme changes.

Set the default theme in frontmatter:

```yaml
---
theme: dark
title: My Report
---
```

## Print and PDF Support

Charts are optimized for printing to PDF:

- **High-Quality Rendering**: Uses SVG renderer for crisp vector graphics at any zoom level
- **No Page Breaks in Charts**: CSS prevents charts and tables from being split across pages
- **All Labels Visible**: Category labels are always shown (with 45° rotation to fit)

When printing reports to PDF, all content stays intact without visual elements being cut off.

## Visual Style (mdsinabox theme)

Clean, data-focused styling:

- **Font**: Helvetica Neue (system sans-serif)
- **Signature**: Orange accent line at top of reports
- **Background**: Paper (`#f8f8f8`) for light, dark (`#231f20`) for dark

### Color Palette

| Color | Hex | Use |
|-------|-----|-----|
| Primary Blue | `#0777b3` | Primary series |
| Secondary Orange | `#bd4e35` | Secondary series, accent |
| Info Blue | `#638CAD` | Tertiary |
| Positive Green | `#2d7a00` | Success, positive values |
| Warning Amber | `#e18727` | Warnings |
| Error Red | `#bc1200` | Errors, negative emphasis |

## Project Structure

```
chart-skill/
├── ts-src/                  # TypeScript implementation
│   ├── cli.ts              # CLI entry point
│   ├── index.ts            # Library exports
│   ├── types.ts            # TypeScript type definitions
│   ├── core/               # Shared utilities
│   │   ├── themes.ts       # Colors, palettes, theme config
│   │   ├── formatting.ts   # Number formatting
│   │   └── css.ts          # CSS generation
│   ├── charts/             # 17 chart type modules
│   │   ├── bar.ts, line.ts, area.ts, pie.ts, scatter.ts, bubble.ts
│   │   ├── boxplot.ts, histogram.ts, waterfall.ts, xmr.ts
│   │   ├── sankey.ts, funnel.ts, heatmap.ts, calendar.ts
│   │   └── sparkline.ts, combo.ts, dumbbell.ts
│   ├── components/         # 8 UI component modules
│   │   ├── big_value.ts, delta.ts, alert.ts, note.ts
│   │   ├── text.ts, textarea.ts, empty_space.ts, table.ts
│   └── layout/             # Report parser
│       ├── parser.ts       # Markdown layout parsing
│       └── templates.ts    # HTML templates
├── build_skill.py           # Builds .skill package for distribution
├── tests/
│   ├── harness/            # Visual test harness markdown
│   ├── dashboard-inline/    # Test dashboard with inline JSON
│   └── dashboard-with-refs/ # Test dashboard with file references
├── docs/
│   ├── MD-CHARTS-PROJECT.md # Original project specification
│   └── agents.md            # Skill authoring reference
└── skill-bundle/            # Source files for the skill
    ├── SKILL.md             # Skill instructions (with YAML frontmatter)
    ├── reference/
    │   └── chart-types.md   # Complete API reference
    └── examples/            # JSON and markdown examples
```

## Warning Messages

The chart generator outputs helpful warnings to stderr when issues are detected:

| Warning | Cause | Solution |
|---------|-------|----------|
| `Invalid JSON in 'bar' block` | Malformed JSON syntax | Check JSON syntax, ensure proper quoting |
| `Unknown component type 'bars'` | Typo in chart type | Use suggested type (e.g., `bar` not `bars`) |
| `Cannot resolve 'file=...'` | File reference without base directory | Use file path argument or inline JSON |
| `Row exceeds 16 columns` | Too many components in one row | Reduce component widths or split into rows |
| `Invalid value for 'value' in big_value` | Wrong data type (e.g., string instead of number) | Ensure values match expected types |

Warnings include context like content previews, suggestions for similar types, and section/row information to help locate issues.

## CLI Options

```bash
npx mviz dashboard.md -o output.html    # Generate HTML (requires -o flag)
npx mviz --lint dashboard.md            # Validate only (no output)
npx mviz -l spec.json                   # Short form of --lint
```

The `-o` flag is required and specifies the output file. The `--lint` flag validates your spec without generating HTML output. Useful for CI/CD pipelines or quick validation.

## Running Tests

```bash
cd ts-src
npm test                    # TypeScript tests (vitest)
npm run build               # Build TypeScript
npm run typecheck           # Type checking only
```

## Skill Bundle

The skill bundle (`skill-bundle-compact/`) is optimized for Claude for Web with minimal token usage (~750 tokens). Supports essential types:
- **Charts:** bar, line, scatter
- **Components:** table (with sparklines), note, textarea, empty_space

For additional chart types (pie, area, heatmap, sankey, etc.), Claude can reference the TypeScript source code in this repository. See `Best_practices.md` for layout guidance and visualization principles.

## Using with Claude

### Claude Code (CLI)

The skill is automatically available when working in this project directory.

### Claude Web (claude.ai)

1. Create a new Claude project
2. Upload the `.skill` file or all files from `skill-bundle/` to the project knowledge base
3. Claude will have access to the skill, examples, and generator

## Dependencies

- Node.js 20+

## Design Philosophy

Data visualization best practices:
- Maximize data-ink ratio (minimal non-data elements)
- Tight, dense layouts for reports
- No gratuitous animations or visual clutter
- Clean, minimal axes (no domain lines, subtle grid)
- Linear interpolation for accurate data representation
- Focus on data clarity over decoration

## License

MIT
