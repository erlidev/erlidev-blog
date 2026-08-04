---
title: "Plotly Chart Demo"
date: 2026-08-04T12:00:00-06:00
draft: true
description: "A test page demonstrating responsive, interactive Plotly charts in Erlidev posts."
summary: "Testing page-bundle and inline Plotly chart definitions with representative LLM benchmark data."
categories:
  - Engineering
tags:
  - Plotly
  - Data Visualization
  - LLM Benchmarks
showToc: true
TocOpen: true
---

This draft tests Erlidev's Plotly shortcode with representative LLM benchmark data. The numbers are synthetic and must not be interpreted as real model results.

## Benchmark comparison

The first chart loads its definition from `benchmark-results.json` in this post's page bundle. Model A has the highest coding and retrieval scores, while Model B leads on reasoning.

{{< plotly src="benchmark-results.json" alt="Grouped bar chart of synthetic Model A and Model B scores. Model A scores 84 in coding, 78 in reasoning, and 91 in retrieval. Model B scores 80, 83, and 86 respectively." />}}

| Task | Model A | Model B |
|---|---:|---:|
| Coding | 84 | 80 |
| Reasoning | 78 | 83 |
| Retrieval | 91 | 86 |

The table provides the same values in a non-interactive format.

## Scaling trend

The second chart defines a smaller data set inline. The synthetic benchmark score rises rapidly between one and seven billion parameters, then improves more slowly at larger sizes.

{{< plotly alt="Line chart of synthetic benchmark score by parameter count. Scores increase from 42 at one billion parameters to 81 at 30 billion parameters, with gains slowing after seven billion parameters." >}}
{
  "data": [
    {
      "type": "scatter",
      "mode": "lines+markers",
      "name": "Synthetic score",
      "x": [1, 3, 7, 13, 30],
      "y": [42, 58, 71, 77, 81],
      "line": { "color": "#636efa", "width": 3 },
      "marker": { "size": 9 },
      "hovertemplate": "%{x}B parameters: %{y}<extra></extra>"
    }
  ],
  "layout": {
    "title": { "text": "Synthetic scaling trend" },
    "xaxis": { "title": { "text": "Parameters (billions)" }, "type": "log" },
    "yaxis": { "title": { "text": "Benchmark score" }, "range": [35, 85] },
    "hovermode": "closest"
  }
}
{{< /plotly >}}

| Parameters | Score |
|---:|---:|
| 1B | 42 |
| 3B | 58 |
| 7B | 71 |
| 13B | 77 |
| 30B | 81 |

## Test checklist

When previewing this post, verify that:

1. Both charts render without browser-console errors.
2. Hover labels display the expected values.
3. Legends and toolbar controls work.
4. Charts resize at narrow viewport widths.
5. Both charts remain readable in the site's light and dark modes.
6. The two accessible data tables contain the same values as the charts.
