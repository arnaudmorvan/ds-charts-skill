# Bar charts

## bar-v.yaml

```yaml
id: bar-v
title: Vertical Bar Chart
size: { width: 432, height: 240 }
data_shape: '[{ label: string, value: number }]'
defaults:
  grid: true
  xAxis: true
  yAxis: true
  cornerRadius: 4    # top of bar
  barWidth: auto     # = (slot * 0.6)
  highlight: max     # null | 'max' | index → bar in chart/positive
geometry:
  padding: { top: 16, right: 12, bottom: 28, left: 36 }
  yTicks: 4
  slotWidth: (width - padL - padR) / N
  barWidth: slotWidth * 0.6
build_order:
  1: gridLines
  2: yLabels
  3: foreach datum → buildBar(slotX + (slotW - barW)/2, scaleY(value), barW, barH, fill)
  4: xLabels
colors:
  bar: chart/1
  highlight: chart/positive  (or chart/2)
  grid: border/subtle
  axis: text/secondary
```

## bar-h.yaml

```yaml
id: bar-h
title: Horizontal Bar Chart
size: { width: 432, height: 280 }
data_shape: '[{ label: string, value: number }]'
defaults:
  cornerRadius: 4    # right end of bar
  showValue: true    # value at bar end
geometry:
  padding: { top: 12, right: 56, bottom: 12, left: 100 }
  rowHeight: (height - padT - padB) / N
  barHeight: rowHeight * 0.55
build_order:
  1: yLabels (left-aligned in left padding, text/secondary 12px)
  2: foreach datum → buildBar(padL, rowY + (rowH - barH)/2, scaleX(value), barH, fill)
       cornerRadius: { tl: 0, tr: 4, bl: 0, br: 4 }
  3: valueLabels (right of bar, text/primary 12px, bold)
colors:
  bar: chart/1
```

## bar-grouped.yaml

```yaml
id: bar-grouped
title: Grouped Bar Chart
size: { width: 432, height: 280 }
data_shape: '[{ label: string, values: [number] }]'  # label = category, values = series
defaults:
  legend: true
  cornerRadius: 3
geometry:
  padding: { top: 16, right: 12, bottom: 44, left: 36 }
  groupWidth: (width - padL - padR) / N
  barWidth: (groupWidth * 0.7) / S    # S = series count
  groupGap: groupWidth * 0.3
build_order:
  1: gridLines
  2: yLabels
  3: foreach group → foreach series s → buildBar(groupX + s*barW, ..., chart/{s+1})
  4: xLabels
  5: legend
notes:
  - Y scale: max of ALL values across series
  - max 4 series
```

## bar-stacked.yaml

```yaml
id: bar-stacked
title: Stacked Bar Chart
size: { width: 432, height: 280 }
data_shape: '[{ label: string, values: [number] }]'  # values = stacked segments
defaults:
  normalize: false   # if true, stacks normalized to 100%
  legend: true
  cornerRadius: 4    # only on top segment
geometry:
  padding: { top: 16, right: 12, bottom: 44, left: 36 }
  slotWidth: (width - padL - padR) / N
  barWidth: slotWidth * 0.6
build_order:
  1: gridLines
  2: yLabels
  3: foreach group → cumY = baseY; foreach segment s (top→bottom of stack):
       segH = ratio * usableH;
       buildBar(slotX + (slotW-barW)/2, cumY - segH, barW, segH, chart/{s+1})
       cornerRadius: top segment = 4 top, bottom segment = 4 bottom (if last only), middle = 0
       cumY -= segH
  4: xLabels
  5: legend
```
