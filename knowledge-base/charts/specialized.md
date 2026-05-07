# Specialized charts

## kpi.yaml

```yaml
id: kpi
title: KPI Card (big number + delta + sparkline)
container: kpi-card  # forced — this IS a card
size: { width: 240, height: auto }
data_shape: '{ value: number, unit?: string, delta: number, trend: [number], label: string }'
defaults:
  showSparkline: true
  deltaFormat: '+0.0%'
build_order:
  1: label (12/500 uppercase tracking-wide, text/secondary)
  2: row [value (32/700) + delta (icon arrow + text 13/500, color=success/danger)]
  3: sparkline (FILL × FIXED 48px) using sparkline.yaml spec
trend_color_logic:
  delta >= 0 → semantic/success / chart/positive
  delta <  0 → semantic/danger  / chart/negative
```

## heatmap-cal.yaml

```yaml
id: heatmap-cal
title: Heatmap Calendar (GitHub-style)
size: { width: 720, height: 140 }
data_shape: '[{ date: "YYYY-MM-DD", value: number }]'
defaults:
  weekStart: 'mon'    # 'mon' | 'sun'
  cellSize: 12
  cellGap: 3
  buckets: 5          # value scale buckets → opacity steps
geometry:
  weeks: ceil(days / 7)
  width: weeks * (cellSize + cellGap)
  rows: 7
build_order:
  1: monthLabels (top, 11px text/secondary, positioned at first week of each month)
  2: weekdayLabels (left, Mon/Wed/Fri only, 11px)
  3: foreach day → rect 12x12 r=2:
       col = weekIndex
       row = (date.weekday - weekStartOffset + 7) % 7
       fill: chart/positive @ opacity = bucket(value) / buckets
       empty cells: border/subtle @ 0.3
  4: legend (bottom-right) "Less ▢▢▢▢▢ More"
colors:
  cell: chart/positive (or chart/1)
  empty: surface/subtle
```

## bubble.yaml

```yaml
id: bubble
title: Bubble Chart
size: { width: 432, height: 280 }
data_shape: '[{ x: number, y: number, r: number, label?: string, group?: string }]'
defaults:
  grid: true
  xAxis: true
  yAxis: true
  rRange: [6, 32]
geometry:
  padding: { top: 12, right: 12, bottom: 28, left: 36 }
build_order:
  1: gridLines (both axes)
  2: yLabels, xLabels
  3: foreach pt → ellipse(scaleX(x), scaleY(y), r, r, chart/{group} @ 0.6 fill, chart/{group} 1.5 stroke)
notes:
  - groups → cycle chart/i
  - if no group, all use chart/1
```

## funnel.yaml

```yaml
id: funnel
title: Funnel Chart
size: { width: 432, height: 320 }
data_shape: '[{ label: string, value: number }]'  # ordered top→bottom
defaults:
  showPct: true       # % of previous stage
  showValue: true
  shape: 'trapezoid'  # 'trapezoid' | 'bar'
geometry:
  padding: { top: 12, right: 100, bottom: 12, left: 100 }
  rowHeight: (height - padT - padB) / N
  maxValue: max(values)
  cx: width / 2
build_order:
  for trapezoid:
    foreach datum i → polygon vertices:
      wTop    = (values[i]   / maxValue) * (width - padL - padR)
      wBottom = (values[i+1] / maxValue) * (width - padL - padR)  // narrower
      vertices: (cx - wTop/2, y), (cx + wTop/2, y),
                (cx + wBottom/2, y + rowHeight), (cx - wBottom/2, y + rowHeight)
      fill: chart/{i+1} or chart/1 with decreasing opacity
  labels:
    left: stage name (text/primary 13/500)
    right: value + (% of prev) (text/secondary 12/400)
```

## radar.yaml

```yaml
id: radar
title: Radar / Spider Chart
size: { width: 280, height: 280 }
data_shape: '[{ axis: string, value: number }] (3-8 axes)'
defaults:
  rings: 4            # concentric grid rings
  scale: 100          # max value
  fillOpacity: 0.25
  showAxisLabels: true
geometry:
  cx, cy: width/2, height/2
  rMax: min(cx, cy) - 28
  N: axes count
  angle(i): -π/2 + i * 2π/N
build_order:
  1: gridRings → N polygon outlines at r = (k/rings) * rMax, stroke=border/subtle, fills=[]
  2: gridSpokes → from cx,cy to (cx + rMax*cos(angle(i)), cy + rMax*sin(angle(i))), stroke=border/subtle
  3: dataPolygon vertices = (cx + (v/scale)*rMax * cos(angle(i)), cy + (v/scale)*rMax * sin(angle(i)))
       fill: chart/1 @ 0.25
       stroke: chart/1 1.5px
  4: dataDots ellipse 8x8 at each vertex (chart/1)
  5: axisLabels at r = rMax + 14 (text/secondary 12px, anchored to angle)
```

## waterfall.yaml

```yaml
id: waterfall
title: Waterfall Chart
size: { width: 432, height: 280 }
data_shape: '[{ label: string, value: number, type: "positive"|"negative"|"total" }]'
defaults:
  grid: true
  showConnectors: true
  cornerRadius: 4
geometry:
  padding: { top: 16, right: 12, bottom: 28, left: 36 }
  slotWidth: (width - padL - padR) / N
  barWidth: slotWidth * 0.6
  cum: 0  // running total
build_order:
  for each datum i:
    if type=='total':
      bar from baseline to value, color=chart/neutral
      cum = value
    else:
      barHeight = |value| based on Y scale
      yStart = scaleY(cum)
      yEnd   = scaleY(cum + value)
      bar from min(yStart,yEnd) height=|yStart-yEnd|, color=chart/positive (pos) or chart/negative (neg)
      cum += value
  connectors: dashed line from top-right of bar i to top-left of bar i+1 (border/subtle)
  xLabels, yLabels
```

## step-progress.yaml

```yaml
id: step-progress
title: Step Progress (horizontal stepper)
size: { width: 432, height: 80 }
data_shape: '[{ label: string, status: "done"|"current"|"todo" }]'
defaults:
  dotSize: 28
  connectorHeight: 2
geometry:
  spacing: (width - dotSize) / (N - 1)
  centerY: 24
build_order:
  1: connectors → for i in 0..N-2:
       line from dotX(i)+dotSize/2 to dotX(i+1)+dotSize/2
       color: if step i is done → chart/positive, else border/subtle
       weight: 2
  2: dots → ellipse 28x28 at each step:
       done    → fill chart/positive, white check icon centered
       current → fill chart/1, white inner dot 8x8 OR ring outline
       todo    → fill surface/subtle, stroke border/default 1.5
  3: labels → text 12/500 below each dot, centered, color text/secondary (todo) / text/primary (current/done)
```
