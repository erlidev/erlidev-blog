# Plotly charts

Erlidev supports interactive Plotly charts through the `plotly` Hugo shortcode. Plotly.js is loaded only on pages containing at least one chart. The integration uses the explicitly pinned Plotly.js 3.6.0 CDN bundle; it does not use the obsolete `plotly-latest` URL.

Chart definitions must be trusted content committed to this repository. Do not pass untrusted user input into Plotly configuration.

## Recommended: load JSON from a page bundle

Store the chart definition beside the post:

```text
content/posts/model-benchmark/
├── index.md
└── benchmark-chart.json
```

Create `benchmark-chart.json`:

```json
{
  "data": [
    {
      "type": "bar",
      "name": "Model A",
      "x": ["Coding", "Reasoning", "Retrieval"],
      "y": [82.4, 76.1, 88.7]
    },
    {
      "type": "bar",
      "name": "Model B",
      "x": ["Coding", "Reasoning", "Retrieval"],
      "y": [79.8, 81.5, 84.2]
    }
  ],
  "layout": {
    "title": { "text": "Benchmark results" },
    "xaxis": { "title": { "text": "Task category" } },
    "yaxis": { "title": { "text": "Score" }, "range": [0, 100] },
    "barmode": "group"
  },
  "config": {
    "displaylogo": false,
    "responsive": true
  }
}
```

Reference it from `index.md`:

```go-html-template
{{</* plotly src="benchmark-chart.json" alt="Grouped bar chart comparing Model A and Model B across coding, reasoning, and retrieval benchmarks" /*/>}}
```

Page-bundle JSON is recommended because it keeps large chart definitions out of the article, provides JSON syntax checking in editors, and makes data changes easier to review in Git.

## Inline JSON

Small charts can be defined directly inside the Markdown file:

```go-html-template
{{</* plotly alt="Line chart showing benchmark score by model size" */>}}
{
  "data": [
    {
      "type": "scatter",
      "mode": "lines+markers",
      "x": [1, 3, 7, 13],
      "y": [42, 58, 71, 79],
      "name": "Benchmark score"
    }
  ],
  "layout": {
    "xaxis": { "title": { "text": "Parameters (billions)" } },
    "yaxis": { "title": { "text": "Score" } }
  }
}
{{</* /plotly */>}}
```

Do not provide both `src` and inline JSON. The Hugo build fails deliberately if both are present, if neither is present, if the JSON is invalid, or if a page-bundle resource cannot be found.

## Figure schema

The top-level JSON object accepts the same arguments used by `Plotly.newPlot`:

- `data`: Required array of Plotly traces.
- `layout`: Optional Plotly layout object.
- `config`: Optional Plotly configuration object.

The integration supplies these defaults when they are omitted:

```json
{
  "layout": {
    "autosize": true
  },
  "config": {
    "displaylogo": false,
    "responsive": true
  }
}
```

Values in the chart JSON override those defaults.

Consult the [official Plotly JavaScript documentation](https://plotly.com/javascript/) for trace types, layout attributes, annotations, hover behavior, and other chart options.

## Accessibility

Always supply a useful `alt` value. It should state the chart type, measured quantities, comparison, and main result where practical. Do not use text such as "chart" or "benchmark graph" without explaining the information presented.

An interactive visualization is not a substitute for reporting the underlying result. The surrounding article should summarize the significant conclusions, and benchmark posts should link to a table, CSV, or other accessible data representation when possible.

If JavaScript is disabled or Plotly's CDN cannot load, the page displays a message indicating that the interactive chart requires JavaScript. Important findings therefore must also appear in the article text.

## Sizing and responsiveness

Charts fill the available article width and have a default minimum height of 24rem. Plotly resizes them when the viewport changes.

Set an explicit height in the chart layout when a visualization needs more room:

```json
{
  "layout": {
    "height": 640
  }
}
```

Avoid fixed widths. They interfere with responsive layouts on phones and narrow browser windows.

## Light and dark modes

Charts automatically follow PaperMod's active light or dark mode. The integration reads PaperMod's CSS variables and applies them to Plotly's:

- Figure and plotting-area backgrounds
- Titles, annotations, legend labels, and hover labels
- Cartesian axis labels, grid lines, and zero lines
- 3D scene backgrounds and axes
- Polar chart backgrounds and axes
- Update menus, sliders, and the mode bar

Changing the site's theme toggle redraws every chart on the page without reloading it. Chart data and configuration remain registered with the page; Plotly may reset transient interactions such as zoom when it redraws.

The site theme intentionally overrides `paper_bgcolor`, `plot_bgcolor`, global font color, and axis colors from chart JSON. This prevents a chart definition with fixed light colors from becoming unreadable in dark mode. Trace colors, marker colors, color scales, annotations, and other visualization-specific choices remain under the chart definition's control. Choose trace colors that have sufficient contrast in both modes.

## Multiple charts

Use the shortcode repeatedly for multiple charts:

```go-html-template
{{</* plotly src="accuracy.json" alt="Accuracy by model" /*/>}}

{{</* plotly src="latency.json" alt="Median response latency by model" /*/>}}
```

Each chart receives a unique element identifier. Plotly.js itself is loaded only once per page.

## Local preview and validation

Start the draft server:

```sh
hugo server --buildDrafts --environment development
```

Check the chart at desktop and mobile widths. Verify hover labels, legend behavior, axis titles, colors in light and dark site modes, and the surrounding written explanation.

Run a production build before committing:

```sh
hugo --gc --minify
```

A malformed or missing chart definition stops the build and reports the affected content file.

## CDN dependency

Pages containing charts request Plotly.js 3.6.0 from `cdn.plot.ly`. Pages without charts make no Plotly request. A pinned version prevents an upstream release from changing existing charts without a repository change.

This arrangement sends a visitor's IP address and request metadata to Plotly's CDN when the visitor opens a chart page. If eliminating that external request becomes necessary, download the pinned Plotly bundle into `static/js/` and change the script URL in `layouts/_shortcodes/plotly.html` to the local path.
