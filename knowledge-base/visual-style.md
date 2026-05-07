# Visual Style — Charts graphiques

Deux styles disponibles : **`graphical`** (polished, showcase) et **`data`** (flat, compact, dashboard).

---

## Style `graphical` (défaut)

Inspiré du [Smart Charts Kit](https://www.figma.com/design/unAQSfuwGyYnv4wQyNTO8h) (Figma Community).

## Principes

1. **Plus aéré, moins miniature** — cards 432×440 par défaut (au lieu de 320×320).
2. **Big number dominant** — tout chart a souvent un nombre principal en 36-56pt en gros au-dessus de la viz.
3. **Pills partout** — légendes, deltas, badges, annotations sont des pills arrondies.
4. **Anneaux décoratifs** — pour donut/gauge/progress, ajouter 2-3 cercles concentriques fins en arrière (opacité 8-15%) pour donner du relief.
5. **Aires gradient** — sous les courbes, gradient vertical (var couleur 25% → 0%) plutôt que solide opacité.
6. **Bars épaisses arrondies** — barres ≥ 24px avec corner radius = bar_width / 2 (pill verticale).
7. **Bouton "Details" en bas** — pill rempli, optionnel, en bas de chaque card.
8. **Padding généreux** — 28-32px (vs 20px standard).

## Spec d'une card "graphique"

```
Card (VERTICAL auto-layout)
├── width: 432 (standard) | 480 (showcase) | FILL si dans grid
├── padding: 28 28 24 28
├── gap: 16
├── cornerRadius: 20  (vs 12 standard)
├── fill: chart/surface
├── stroke: chart/border (1px, opacity 0.6)
├── shadow: 0 4px 16px rgba(0,0,0,0.04)
└──
   ├── Header (HUG, gap=4)
   │  ├── Eyebrow (text 11/600 uppercase tracking=1, chart/text/secondary)
   │  ├── Title (text 17/700, chart/text/primary)
   │  └── Subtitle (text 13/400, chart/text/secondary, optional)
   │
   ├── BigNumber row (HORIZONTAL, baseline-aligned, gap=10, optional)
   │  ├── Value (text 44/700, chart/text/primary, e.g. "$48.2k")
   │  └── DeltaPill (HORIZONTAL pill, padding=8/4, radius=999, fill=positive@12%)
   │     ├── arrow (text 12/700, positive)
   │     └── delta (text 12/600, positive, e.g. "+12.4%")
   │
   ├── Chart canvas (FILL × FIXED, height varies by type)
   │  └── ... geometry with SCALE constraints
   │
   ├── Legend row (HORIZONTAL wrap, gap=10, optional)
   │  └── LegendPill × N
   │
   └── Details button (HORIZONTAL, FILL × HUG, padding=14/0, radius=14, optional)
      ├── fill: chart/surface
      ├── stroke: chart/border
      └── label "Details" (13/600, chart/text/primary, centered)
```

## Tailles canvas par type

| Chart | Card size | Canvas height |
|-------|-----------|---------------|
| line, area, multi-line | 432 × 440 | 200 |
| bar-v, bar-grouped, bar-stacked | 432 × 440 | 220 |
| bar-h | 432 × 440 | 220 |
| sparkline (in card) | 432 × 240 | 80 |
| donut | 432 × 480 | 280 |
| pie | 432 × 480 | 280 |
| progress-ring | 432 × 440 | 240 |
| multi-ring | 432 × 480 | 280 |
| gauge | 432 × 400 | 200 |
| kpi | 432 × 280 | 100 |
| heatmap-cal | 480 × 280 | 160 |
| bubble | 432 × 440 | 240 |
| funnel | 432 × 480 | 300 |
| radar | 432 × 480 | 280 |
| waterfall | 480 × 440 | 240 |
| step-progress | 432 × 480 | 320 |

## Règles graphical par famille de chart

### Line / Area / Multi-line / Sparkline
- **Area gradient obligatoire** : gradient vertical `chart/series/1 @ 35%` → `@ 0%`
- **Dots** sur la série principale (ellipse 10×10, fill=surface, stroke=chart/series/1 2.5px)
- **Grid** lignes fines `chart/grid` (4 lignes horizontales)
- **Multi-line** : série 2 en dashed `[4,4]`, série 3+ en `[2,4]`
- Sparkline in-card : hauteur 80, pas de grid, pas de labels

### Bar-v / Bar-grouped / Bar-stacked
- **Barres pilule** : barW ≥ 24px, cornerRadius = barW/2 (pill verticale complète)
- **Track** : rectangle plein hauteur à `chart/track @ 0.55` derrière chaque barre
- **Max highlight** : la barre max à opacité 1, les autres à 0.55
- Bar-grouped : barres plus fines (~18px), toujours track+pill
- Bar-stacked : segment du bas arrondi en bas, segment du haut arrondi en haut, les autres cornerRadius=0

### Bar-h
- Barres horizontales pilule (cornerRadius = barH/2)
- Track pleine largeur à `chart/track @ 0.55`
- Valeur à droite de chaque barre (text/primary, Bold 12px)

### Donut / Pie
- **Anneaux décoratifs** : 3 cercles concentriques fins (`chart/series/1 @ 5-9%`) autour du donut
- Valeur centrale en Bold 32pt, label en Semibold 11pt uppercase tracking=1
- Pie : pas d'anneaux décoratifs

### Progress-ring / Multi-ring
- Anneaux décoratifs (identique donut)
- Track en `chart/track` (plein anneau)
- Valeur centrale Bold 36pt

### Gauge
- Half-donut, bottom-anchored
- Track en `chart/track`
- Valeur centrale Bold 36pt, alignée horizontalement
- Label min/max aux extremités (text/secondary 11pt)
- Pas d'anneaux décoratifs (car semi-circulaire)

### KPI
- Big number 40pt Bold
- Delta pill (▲/▼ + %, fond positive/negative @ 14%)
- Sparkline gradient sous la courbe

### Heatmap-cal
- Cellules 12×12 r=2, gap=3
- Palette : `chart/positive` @ opacité 0, 0.2, 0.4, 0.7, 1.0 selon bucket
- Légère ombre sur les cellules du top bucket

### Bubble
- Bulles semi-transparentes (fill @ 0.5, stroke @ 1.0)
- Grid lines fins `chart/grid`
- Groupes → `chart/series/1..N`

### Funnel
- Gradient de couleur du haut vers le bas : `chart/series/1` → `chart/series/3` (opacity 1.0 → 0.45)
- Labels flottants à gauche (stage name) et à droite (value + %) en text/secondary
- Coins arrondis : cornerRadius=6 sur chaque segment

### Radar
- Grille polygon en `chart/grid`
- Polygone data : fill `chart/series/1 @ 0.25`, stroke `chart/series/1 2px`
- Dots sur chaque axe : ellipse 8×8 fill=surface, stroke=chart/series/1 2px
- Labels axes à rMax+14

### Waterfall
- Positif → `chart/positive`, négatif → `chart/negative`, total → `chart/neutral`
- Barres pilule en haut (cornerRadius = barW/2 en haut seulement)
- Connecteurs en pointillés `chart/grid [4,4]`

### Step-progress
- Dots 28×28 ; done → fill `chart/positive`, current → fill `chart/series/1`, todo → fill `chart/surface-bg` + stroke `chart/border`
- Connecteurs entre dots : done → `chart/positive`, pending → `chart/grid`
- Labels 12pt Medium sous chaque dot

---

## Composants visuels réutilisables

### Pill

```js
function pill(parent, {label, fillVar, textVar, opacity=1, icon=null, fontSize=12}) {
  const f = figma.createFrame();
  parent.appendChild(f);
  f.layoutMode = 'HORIZONTAL';
  f.primaryAxisSizingMode = 'AUTO';
  f.counterAxisSizingMode = 'AUTO';
  f.paddingLeft = 10; f.paddingRight = 10;
  f.paddingTop = 4; f.paddingBottom = 4;
  f.itemSpacing = 4;
  f.cornerRadius = 999;
  f.fills = [mkFill(fillVar, opacity)];
  if (icon) {
    const i = figma.createText(); f.appendChild(i);
    i.fontName = F.bold; i.fontSize = fontSize;
    i.characters = icon; i.fills = [mkFill(textVar)];
  }
  const t = figma.createText(); f.appendChild(t);
  t.fontName = F.sb; t.fontSize = fontSize;
  t.characters = label; t.fills = [mkFill(textVar)];
  return f;
}
```

### Decorative rings (background pour radial charts)

```js
function decorativeRings(parent, cx, cy, radii, colorVar) {
  // 3 anneaux fins concentriques en bg, opacité décroissante
  radii.forEach((r, i) => {
    const e = figma.createEllipse();
    parent.appendChild(e);
    e.x = cx - r; e.y = cy - r; e.resize(r * 2, r * 2);
    e.fills = [];
    e.strokes = [mkFill(colorVar, 0.06 + i * 0.02)];
    e.strokeWeight = 1;
  });
}
```

### Area gradient (under line)

```js
function areaGradient(fillVar) {
  // GRADIENT_LINEAR top→bottom, var color at 25% top, transparent bottom
  const paint = {
    type: 'GRADIENT_LINEAR',
    gradientTransform: [[0, 1, 0], [-1, 0, 1]],
    gradientStops: [
      { position: 0, color: { r: 0, g: 0, b: 0, a: 0.25 } },
      { position: 1, color: { r: 0, g: 0, b: 0, a: 0 } },
    ],
  };
  // Bind first stop to chart variable
  return figma.variables.setBoundVariableForPaint(paint, 'color', fillVar);
}
```

### Big value + delta pill

```js
function bigValueRow(parent, value, delta, deltaIsPositive) {
  const row = figma.createFrame();
  parent.appendChild(row);
  row.layoutMode = 'HORIZONTAL';
  row.primaryAxisSizingMode = 'AUTO';
  row.counterAxisSizingMode = 'AUTO';
  row.counterAxisAlignItems = 'CENTER'; // baseline approximation
  row.itemSpacing = 10;
  row.fills = [];
  const v = figma.createText(); row.appendChild(v);
  v.fontName = F.bold; v.fontSize = 44; v.characters = value;
  v.fills = [mkFill(roles.text_primary)];
  if (delta) {
    pill(row, {
      label: (deltaIsPositive ? '+' : '') + delta,
      fillVar: deltaIsPositive ? roles.positive : roles.negative,
      textVar: deltaIsPositive ? roles.positive : roles.negative,
      opacity: 0.12,
      icon: deltaIsPositive ? '▲' : '▼',
      fontSize: 12,
    });
    // re-apply text color (pill bg is opacity 0.12, text full)
  }
  return row;
}
```

## Couleurs : usage cohérent

- **1 série** → `series.1` (couleur primaire)
- **2 séries comparables** → `series.1` + `series.2`
- **3+ séries catégorielles** → cycle `series.1..N`
- **Sentiment** → `positive` / `negative` / `warning` (jamais cycler la palette)
- **Funnel** → `series.1` à 100%, puis chaque stage à -10% opacité descendant
- **Stacked** → cycle `series.1..N` sur les segments
- **Heatmap** → `series.1` avec opacité variable (0.08, 0.25, 0.45, 0.7, 1.0)

## À éviter

- ❌ Couleurs en hex direct dans les fills
- ❌ Cards minuscules (<320px de large)
- ❌ Bars fines (<8px) sans radius

---

## Style `data` (flat / compact / dashboard)

Adapté aux dashboards denses : même données, rendu épuré et compact.

### Spec d'une card `data`

```
Card (VERTICAL auto-layout)
├── width: 380 (fixe)
├── height: AUTO (hugs content)
├── padding: 16 all
├── gap: 10
├── cornerRadius: 10
├── fill: #FFFFFF
├── stroke: #EDEDF2 (1px)
├── shadow: aucune
└──
   ├── Title (text 12/Semi Bold, #1A1A2E)
   ├── [BigNumber + DeltaPill] (HORIZONTAL, optionnel — KPI/progress/gauge seulement)
   │   ├── Value (text 20-24/Bold, #1A1A2E)
   │   └── DeltaPill (pill, cornerRadius=4, fill=positive@10%)
   │       └── label (text 10-11/Medium, positive)
   ├── Chart canvas (layoutAlign=STRETCH, height ~100-160px, layoutMode=NONE)
   │   └── geometry — SCALE constraints sur tous les enfants
   └── Legend (HORIZONTAL wrap, gap=10, optionnel)
       └── dot (6×6, cornerRadius=3) + label (text 10/Regular, #6B6B80)
```

### Palette hex directe (style `data`)

Pas de liaison variable DS — couleurs hex fixes pour garantir la cohérence cross-fichier :

```
s1 #396BF9   s2 #F0AC4C   s3 #5BC59A   s4 #E26AA1   s5 #7E5BE7   s6 #4CCBE0
pos #029341  neg #D6362F  warn #F3A649  neu #9AA3B2
surf #FFFFFF  bg #F5F5FA  brd #EDEDF2
tp #1A1A2E   ts #6B6B80   trk #EEF0F3
```

### Règles `data` vs `graphical`

| Élément | `graphical` | `data` |
|---------|-------------|--------|
| Largeur card | 432–480px | 380px |
| Padding | 28px | 16px |
| Corner radius | 20px | 10px |
| Ombre | `0 4px 16px rgba(0,0,0,0.04)` | aucune |
| Big number | 44pt Bold | 20–24pt Bold (si présent) |
| Chart height | 200–320px | 100–160px |
| Area fill | gradient var DS | gradient hex (s1 @15%) |
| Barres | pill (radius=barW/2), track | rectangle simple, radius 3px (top only) |
| Grid | lignes DS var | `figma.createLine()` stroke `#EDEDF2` |
| Couleurs | variables DS | hex directs (palette ci-dessus) |
| Légende | LegendPill (pill arrondie) | dot 6×6 + texte plat |
| Bouton Details | ✅ optionnel | ❌ absent |
- ❌ Légendes texte sans pill ni dot coloré
- ❌ Donut sans valeur centrale (sauf demande explicite)
- ❌ Plus de 6 séries dans un line chart (illisible)
