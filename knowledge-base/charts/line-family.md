# Line / Area / Sparkline charts

## line.yaml

```yaml
id: line
title: Line Chart
size: { width: 432, height: 220 }
data_shape: '[number] | { labels: [string], values: [number] }'
defaults:
  smooth: true        # cubic-bezier interpolation
  dots: false         # show dot at each point
  area: false         # fill area under line
  grid: true          # horizontal grid lines (4 ticks)
  xAxis: true
  yAxis: true
geometry:
  padding: { top: 16, right: 12, bottom: 28, left: 36 }
  yTicks: 4
  xLabelEvery: 1      # show every Nth label (auto-skip if width<320)
build_order:
  1: gridLines (border/subtle, dashed)
  2: yLabels (text/secondary, 11px)
  3: areaPath (optional, chart/1 @ 0.2 opacity, gradient to transparent)
  4: linePath (chart/1, 2px, ROUND caps)
  5: dots (optional, ellipse 8x8, chart/1, white inner ring 2px)
  6: xLabels (text/secondary, 11px)
colors:
  line: chart/1
  area: chart/1
  dots: chart/1
  grid: border/subtle
  axis: text/secondary
```

## area.yaml

```yaml
id: area
title: Area Chart
extends: line
defaults:
  smooth: true
  area: true          # ← always true
  dots: false
  fillOpacityTop: 0.35
  fillOpacityBottom: 0
```

## multi-line.yaml

```yaml
id: multi-line
title: Multi-Line Chart
size: { width: 432, height: 240 }
data_shape: '[{ label: string, values: [number] }]'
defaults:
  smooth: true
  dots: false
  legend: true
geometry:
  padding: { top: 16, right: 12, bottom: 44, left: 36 }
  yTicks: 4
  legendBelow: true
build_order:
  1: gridLines
  2: yLabels
  3: foreach series → linePath (chart/{i+1})
  4: legend (HORIZONTAL, gap=12, dot 8x8 + label 12px)
notes:
  - common Y scale across all series (min/max across all values)
  - max 6 series (warn user if more)
  - cycle chart/1..chart/N variables
```

## sparkline.yaml

```yaml
id: sparkline
title: Sparkline (mini line, no axes)
size: { width: 200, height: 48 }
data_shape: '[number]'
defaults:
  smooth: true
  area: true
  dots: false
  grid: false
  xAxis: false
  yAxis: false
geometry:
  padding: { top: 4, right: 4, bottom: 4, left: 4 }
build_order:
  1: areaPath (chart/1 @ 0.25, gradient down)
  2: linePath (chart/1, 1.5px, ROUND)
  3: lastDot (ellipse 6x6 at last point, chart/1)
notes:
  - used inside kpi-card or table cells
  - no labels, no grid — context provided by parent
```
