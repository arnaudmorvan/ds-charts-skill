# Radial / Donut / Ring charts

## donut.yaml

```yaml
id: donut
title: Donut Chart
size: { width: 280, height: 280 }
data_shape: '[{ label: string, value: number }]'
defaults:
  innerRatio: 0.62      # rInner / rOuter
  centerValue: auto     # auto = sum, or string, or null
  centerLabel: 'Total'
  legend: true          # vertical right or below
  legendPosition: right
  gap: 0.01             # radians between slices (visual separator)
geometry:
  cx: width / 2 (or width * 0.35 if legend right)
  cy: height / 2
  rOuter: min(cx, cy) - 16
  rInner: rOuter * innerRatio
build_order:
  1: foreach slice → donutSlice(arcData, chart/{i+1})
  2: centerValue text (32/700, text/primary, centered absolute)
  3: centerLabel text (12/500, text/secondary, below value)
  4: legend (VERTICAL, gap=8, dot 10x10 + label 13px + value 13px text/secondary)
colors_palette:
  - chart/1, chart/2, chart/3, chart/4, chart/5  (cycle)
notes:
  - if 1 slice = 100% → just a ring (no separators)
  - sort slices descending by value (optional, default true)
```

## pie.yaml

```yaml
id: pie
title: Pie Chart
extends: donut
defaults:
  innerRatio: 0       # ← solid pie
  centerValue: null
  centerLabel: null
```

## progress-ring.yaml

```yaml
id: progress-ring
title: Progress Ring (single value)
size: { width: 200, height: 200 }
data_shape: 'number (0-100)'
defaults:
  innerRatio: 0.78
  centerValue: '{value}%'
  centerLabel: null
  trackColor: border/subtle
  fillColor: chart/positive   (or chart/1)
geometry:
  cx, cy, rOuter, rInner: same as donut
build_order:
  1: trackRing → donutSlice(0, 2π, trackColor)
  2: progressArc → donutSlice(-π/2, -π/2 + (value/100)*2π, fillColor)
  3: centerValue (32/700)
notes:
  - value > 100 → over-rotation (multi-loop), color shifts to chart/3
```

## multi-ring.yaml

```yaml
id: multi-ring
title: Activity Rings (Apple-style)
size: { width: 220, height: 220 }
data_shape: '[{ label: string, value: number }] (1-4 rings)'
defaults:
  ringWidth: 16
  gap: 4              # px between concentric rings
geometry:
  cx, cy: width / 2
  rMax: min(cx, cy) - 8
  for ring i (outer to inner):
    rOuter_i = rMax - i * (ringWidth + gap)
    rInner_i = rOuter_i - ringWidth
build_order:
  foreach ring i:
    1: trackRing (color = chart/{i+1} @ opacity 0.2)
    2: progressArc (color = chart/{i+1}, from -π/2, sweep = clamp(value,0,100)/100 * 2π)
  legend: HORIZONTAL gap=12, dot + label below
colors:
  ring 1: chart/1 (Move = red typically)
  ring 2: chart/2 (Exercise = green)
  ring 3: chart/3 (Stand = blue)
```

## gauge.yaml

```yaml
id: gauge
title: Gauge / Half-Donut
size: { width: 280, height: 180 }
data_shape: '{ value: number, min?: number, max?: number, unit?: string }'
defaults:
  min: 0
  max: 100
  innerRatio: 0.7
  thresholds: null    # [{ at: 60, color: chart/warning }, { at: 80, color: chart/positive }]
  showMinMax: true
geometry:
  cx: width / 2
  cy: height * 0.85   # bottom-anchored
  rOuter: min(cx, cy * 1.15) - 12
  rInner: rOuter * innerRatio
  startAngle: -π      # left
  endAngle: 0         # right
build_order:
  1: trackArc (border/subtle)
  2: valueArc (chart/1 or threshold-colored, sweep = value/(max-min) * π)
  3: needle (optional thin triangle from center)
  4: bigValue text (32/700, centered horizontally, y = cy - 24)
  5: unit text (13/500, text/secondary, below value)
  6: minLabel left, maxLabel right (12/400, text/secondary, y = cy + 12)
```
