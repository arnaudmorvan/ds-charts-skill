# Data Generators (mode batch)

Snippets JS prêts à coller dans `use_figma`. Tous déterministes via seed optionnel.

## Helpers

```js
function rand(min, max) { return min + Math.random() * (max - min); }
function randInt(min, max) { return Math.floor(rand(min, max + 1)); }
function pick(arr) { return arr[randInt(0, arr.length - 1)]; }

const LABELS = {
  weekday: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
  month:   ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'],
  quarter: ['Q1', 'Q2', 'Q3', 'Q4'],
  segments: ['Mobile', 'Desktop', 'Tablet', 'Other'],
  products: ['Product A', 'Product B', 'Product C', 'Product D'],
  channels: ['Direct', 'Organic', 'Paid', 'Referral', 'Social'],
};
```

## Random walk (line, area, sparkline)

```js
function randomWalk(n = 12, start = 50, drift = 0, vol = 8) {
  const out = [start];
  for (let i = 1; i < n; i++) {
    const step = drift + (Math.random() - 0.5) * vol;
    out.push(Math.max(0, out[i - 1] + step));
  }
  return out.map(v => Math.round(v));
}
// Usage: randomWalk(7, 100, 2, 15)  → 7 days trending up
```

## Bar (vertical/horizontal)

```js
function barData(labels = LABELS.weekday, min = 20, max = 90) {
  return labels.map(l => ({ label: l, value: randInt(min, max) }));
}
```

## Grouped / Stacked bar

```js
function multiSeriesBar(labels, seriesNames, min = 10, max = 60) {
  return seriesNames.map(name => ({
    label: name,
    values: labels.map(() => randInt(min, max)),
  }));
}
// Usage: multiSeriesBar(LABELS.month.slice(0,6), ['Revenue', 'Costs'])
```

## Donut / Pie (Dirichlet-like)

```js
function donutSlices(n = 4, labels = LABELS.segments) {
  const raw = Array.from({ length: n }, () => Math.random() + 0.2);
  const sum = raw.reduce((a, b) => a + b, 0);
  return raw.map((v, i) => ({
    label: labels[i] || `Segment ${i + 1}`,
    value: Math.round((v / sum) * 100),
  }));
}
```

## Progress / Gauge

```js
function progressValue() { return randInt(20, 95); }
```

## Multi-ring (Apple Activity)

```js
function multiRing() {
  return [
    { label: 'Move',    value: randInt(40, 110) },
    { label: 'Exercise', value: randInt(40, 110) },
    { label: 'Stand',   value: randInt(40, 110) },
  ];
}
```

## KPI

```js
function kpiData(unit = '$') {
  const value = randInt(1000, 100000);
  const delta = +(rand(-25, 45)).toFixed(1);
  const trend = randomWalk(12, value * 0.8, value * 0.02, value * 0.05);
  return { value, unit, delta, trend };
}
```

## Heatmap calendar (1 an)

```js
function heatmapYear() {
  const out = [];
  const start = new Date();
  start.setDate(start.getDate() - 365);
  for (let i = 0; i < 365; i++) {
    const d = new Date(start);
    d.setDate(start.getDate() + i);
    // exponential decay → most days low, few days high
    const v = Math.floor(-Math.log(1 - Math.random()) * 5);
    out.push({ date: d.toISOString().slice(0, 10), value: v });
  }
  return out;
}
```

## Funnel

```js
function funnelData() {
  const stages = ['Visits', 'Signups', 'Trials', 'Paid', 'Renewed'];
  let v = randInt(8000, 12000);
  return stages.map(label => {
    const out = { label, value: v };
    v = Math.round(v * rand(0.4, 0.75));
    return out;
  });
}
```

## Radar

```js
function radarData(axes = ['Speed', 'Power', 'Range', 'Agility', 'Stamina', 'Skill']) {
  return axes.map(a => ({ axis: a, value: randInt(30, 95) }));
}
```

## Bubble

```js
function bubbleData(n = 12) {
  return Array.from({ length: n }, (_, i) => ({
    x: rand(0, 100),
    y: rand(0, 100),
    r: rand(8, 32),
    label: `P${i + 1}`,
  }));
}
```

## Waterfall

```js
function waterfallData() {
  const out = [{ label: 'Start', value: randInt(800, 1200), type: 'total' }];
  let cum = out[0].value;
  ['Q1', 'Q2', 'Q3', 'Q4'].forEach(q => {
    const v = randInt(-200, 400);
    out.push({ label: q, value: v, type: v >= 0 ? 'positive' : 'negative' });
    cum += v;
  });
  out.push({ label: 'End', value: cum, type: 'total' });
  return out;
}
```

## Step progress

```js
function stepData() {
  const all = ['Email', 'Profile', 'Bank', 'Verify', 'Done'];
  const cur = randInt(1, all.length - 1);
  return all.map((l, i) => ({
    label: l,
    status: i < cur ? 'done' : i === cur ? 'current' : 'todo',
  }));
}
```
