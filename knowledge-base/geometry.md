# Geometry Helpers

Snippets JS pour `use_figma`. Tous travaillent en coordonnées **locales au chart frame** (origine top-left = 0,0).

## Scaling

```js
// values[] → pixel positions in [0, width]
function scaleX(i, n, width, padL = 0, padR = 0) {
  const usable = width - padL - padR;
  return padL + (n === 1 ? usable / 2 : (i / (n - 1)) * usable);
}

// value → pixel y (inverted: high values = small y)
function scaleY(v, min, max, height, padT = 0, padB = 0) {
  const usable = height - padT - padB;
  if (max === min) return padT + usable / 2;
  return padT + usable - ((v - min) / (max - min)) * usable;
}
```

## Polyline (line chart)

Construction via `figma.createVector()` + `setVectorNetworkAsync`.

```js
async function buildPolyline(parent, points, strokeVar, strokeWidth = 2) {
  const v = figma.createVector();
  parent.appendChild(v);
  v.x = 0; v.y = 0;
  v.resize(parent.width, parent.height);

  const network = {
    vertices: points.map(p => ({ x: p.x, y: p.y })),
    segments: points.slice(0, -1).map((_, i) => ({ start: i, end: i + 1 })),
    regions: [],
  };
  await v.setVectorNetworkAsync(network);

  const stroke = { type: 'SOLID', color: { r: 0, g: 0, b: 0 } };
  v.strokes = [figma.variables.setBoundVariableForPaint(stroke, 'color', strokeVar)];
  v.strokeWeight = strokeWidth;
  v.strokeCap = 'ROUND';
  v.strokeJoin = 'ROUND';
  v.fills = [];
  return v;
}
```

## Smooth curve (cubic bezier through points)

Catmull-Rom → Bezier conversion :

```js
function catmullRomToBezier(points, tension = 0.5) {
  const segs = [];
  for (let i = 0; i < points.length - 1; i++) {
    const p0 = points[i - 1] || points[i];
    const p1 = points[i];
    const p2 = points[i + 1];
    const p3 = points[i + 2] || p2;
    const c1 = { x: p1.x + (p2.x - p0.x) / 6 * tension, y: p1.y + (p2.y - p0.y) / 6 * tension };
    const c2 = { x: p2.x - (p3.x - p1.x) / 6 * tension, y: p2.y - (p3.y - p1.y) / 6 * tension };
    segs.push({ start: i, end: i + 1, tangentStart: { x: c1.x - p1.x, y: c1.y - p1.y },
                                       tangentEnd:   { x: c2.x - p2.x, y: c2.y - p2.y } });
  }
  return segs;
}
```

## Area path (closed polyline + base line)

```js
async function buildArea(parent, points, height, fillVar, opacity = 0.2) {
  const v = figma.createVector();
  parent.appendChild(v);
  v.x = 0; v.y = 0;
  v.resize(parent.width, parent.height);

  const closed = [
    ...points,
    { x: points[points.length - 1].x, y: height },
    { x: points[0].x, y: height },
  ];
  const network = {
    vertices: closed.map(p => ({ x: p.x, y: p.y })),
    segments: closed.map((_, i) => ({ start: i, end: (i + 1) % closed.length })),
    regions: [{
      windingRule: 'NONZERO',
      loops: [closed.map((_, i) => i)],
    }],
  };
  await v.setVectorNetworkAsync(network);
  const fill = { type: 'SOLID', color: { r: 0, g: 0, b: 0 }, opacity };
  v.fills = [figma.variables.setBoundVariableForPaint(fill, 'color', fillVar)];
  v.strokes = [];
  return v;
}
```

## Donut slice (arc)

Donut slices can be built via `createEllipse()` + `arcData`:

```js
function donutSlice(parent, cx, cy, rOuter, rInner, startAngle, endAngle, fillVar) {
  const e = figma.createEllipse();
  parent.appendChild(e);
  e.x = cx - rOuter; e.y = cy - rOuter;
  e.resize(rOuter * 2, rOuter * 2);
  e.arcData = {
    startingAngle: startAngle,        // radians
    endingAngle: endAngle,
    innerRadius: rInner / rOuter,     // 0..1
  };
  const fill = { type: 'SOLID', color: { r: 0, g: 0, b: 0 } };
  e.fills = [figma.variables.setBoundVariableForPaint(fill, 'color', fillVar)];
  e.strokes = [];
  return e;
}

// Build full donut from slices
function buildDonut(parent, slices, cx, cy, rOuter, rInner, vars) {
  const total = slices.reduce((a, s) => a + s.value, 0);
  let acc = -Math.PI / 2; // start at top
  slices.forEach((s, i) => {
    const sweep = (s.value / total) * Math.PI * 2;
    donutSlice(parent, cx, cy, rOuter, rInner, acc, acc + sweep, vars[i % vars.length]);
    acc += sweep;
  });
}
```

## Bars

```js
function buildBar(parent, x, y, w, h, fillVar, radius = 4) {
  const r = figma.createRectangle();
  parent.appendChild(r);
  r.x = x; r.y = y; r.resize(w, h);
  r.topLeftRadius = radius;
  r.topRightRadius = radius;
  r.bottomLeftRadius = 0;
  r.bottomRightRadius = 0;
  const fill = { type: 'SOLID', color: { r: 0, g: 0, b: 0 } };
  r.fills = [figma.variables.setBoundVariableForPaint(fill, 'color', fillVar)];
  r.strokes = [];
  return r;
}
```

## Grid lines

```js
function buildGrid(parent, ticks, width, height, padT, padB, lineVar) {
  const usable = height - padT - padB;
  ticks.forEach(t => {
    const y = padT + usable - t.ratio * usable;
    const line = figma.createLine();
    parent.appendChild(line);
    line.x = 0; line.y = y;
    line.resize(width, 0);
    const stroke = { type: 'SOLID', color: { r: 0, g: 0, b: 0 }, opacity: 0.4 };
    line.strokes = [figma.variables.setBoundVariableForPaint(stroke, 'color', lineVar)];
    line.strokeWeight = 1;
    line.dashPattern = [2, 4];
  });
}
```

## Axis labels

Standard text nodes positioned absolutely in the chart frame :

```js
function buildXLabels(parent, labels, width, height, padL, padR, padB, font, color) {
  labels.forEach((l, i) => {
    const t = figma.createText();
    parent.appendChild(t);
    t.fontName = font;
    t.fontSize = 11;
    t.characters = String(l);
    const x = scaleX(i, labels.length, width, padL, padR);
    t.x = x - t.width / 2;
    t.y = height - padB + 6;
    const fill = { type: 'SOLID', color: { r: 0, g: 0, b: 0 } };
    t.fills = [figma.variables.setBoundVariableForPaint(fill, 'color', color)];
  });
}
```

## Nice ticks (Y axis)

```js
function niceTicks(min, max, count = 4) {
  const range = max - min;
  const raw = range / count;
  const pow = Math.pow(10, Math.floor(Math.log10(raw)));
  const norm = raw / pow;
  const nice = norm < 1.5 ? 1 : norm < 3 ? 2 : norm < 7 ? 5 : 10;
  const step = nice * pow;
  const niceMin = Math.floor(min / step) * step;
  const niceMax = Math.ceil(max / step) * step;
  const ticks = [];
  for (let v = niceMin; v <= niceMax + step / 2; v += step) ticks.push(v);
  return { ticks, min: niceMin, max: niceMax };
}
```

## Format

```js
function fmt(v) {
  if (Math.abs(v) >= 1e6) return (v / 1e6).toFixed(1) + 'M';
  if (Math.abs(v) >= 1e3) return (v / 1e3).toFixed(1) + 'k';
  return String(Math.round(v));
}
```
