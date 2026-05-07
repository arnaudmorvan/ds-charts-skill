---
name: ds-charts-skill
description: "Génère des graphiques Figma (line, bar, donut, gauge, sparkline, heatmap, radar, funnel, KPI…) via MCP. USE WHEN: l'utilisateur veut créer un chart/graphique pour un dashboard, en standalone ou dans une card, avec des données précises (chart unitaire) ou un batch d'exemples (données aléatoires). Utilise les variables couleur du Design System et auto-layout responsive."
argument-hint: "Type de graphique + container (card/bare/kpi) + style (graphical/data) + données ou 'batch N'"
---

# Skill — Figma Charts Builder

> Génère des graphiques de dashboard dans Figma via MCP. Parametric, responsive (auto-layout), couleurs liées aux **variables du DS**. Inspiré du [Smart Charts Kit](https://www.figma.com/design/unAQSfuwGyYnv4wQyNTO8h/31-Free-Smart--Charts-Kit) (référence visuelle).

## Quand utiliser ce skill

- L'utilisateur demande de **créer un graphique / chart** pour un dashboard
- L'utilisateur veut une **carte KPI** (stat + delta + sparkline)
- L'utilisateur veut **plusieurs exemples de charts** ("génère un batch", "fais-moi 12 graphiques", "remplis cette page de charts")
- L'utilisateur référence un **DS Figma** existant et veut que les couleurs s'y conforment

## Routing — Mode d'exécution

| Demande | Mode | Données |
|--------|------|---------|
| "fais-moi un line chart avec 10, 20, 15, 30, 25" | **single** | fournies par l'utilisateur |
| "génère un donut chart Sales avec 60% A, 40% B" | **single** | fournies |
| "ajoute un graphique progress 75%" | **single** | fournie |
| "génère 10 charts d'exemple" / "remplis la page" | **batch** | aléatoires |
| "un de chaque type" | **batch-showcase** | aléatoires (1 par type) |

## Style visuel — `graphical` vs `data`

| Valeur | Alias acceptés | Description |
|--------|---------------|-------------|
| `graphical` (défaut) | `polished`, `illustré`, `card illustrée` | Card grande (432px+), big number dominant, gradient sous courbe, barres pill, ombres, coins 20px, padding 28px. Idéal pour présentation / showcase / LinkedIn. |
| `data` | `flat`, `compact`, `minimal`, `dashboard` | Card compacte (380px, ~140px de viz), fond blanc, bordure 1px `#EDEDF2`, no shadow, coins 10px, padding 16px. Idéal pour intégration dashboard dense. |

**Détection automatique** dans le prompt :
- Mots-clés → `graphical` : "polished", "illustré", "showcase", "belle card", "poster"
- Mots-clés → `data` : "compact", "flat", "dashboard", "minimal", "dense", "data"
- Aucun mot-clé → défaut `graphical`

> Spec complète des deux styles : [knowledge-base/visual-style.md](./knowledge-base/visual-style.md)

## Container — Comment l'envelopper

| Mode | Description | Cas d'usage |
|------|-------------|-------------|
| `bare` | Chart seul, frame transparent | Insertion dans un layout existant |
| `card` (défaut) | Card DS : header (title + subtitle) + chart + footer optionnel | Dashboard standard |
| `kpi-card` | Big number + delta + mini sparkline | Top row dashboard |

Si l'utilisateur ne précise pas, **défaut = `card`**.

## Catalogue des charts supportés

| ID | Type | Données attendues | YAML |
|----|------|-------------------|------|
| `line` | Line chart simple | `[number]` ou `{labels, values}` | `line.yaml` |
| `multi-line` | Plusieurs séries | `[{label, values: [number]}]` | `multi-line.yaml` |
| `area` | Aire remplie sous courbe | `[number]` | `area.yaml` |
| `bar-v` | Barres verticales | `[{label, value}]` | `bar-v.yaml` |
| `bar-h` | Barres horizontales | `[{label, value}]` | `bar-h.yaml` |
| `bar-grouped` | Barres groupées (séries) | `[{label, values: [number]}]` | `bar-grouped.yaml` |
| `bar-stacked` | Barres empilées | `[{label, values: [number]}]` | `bar-stacked.yaml` |
| `donut` | Donut + valeur centrale | `[{label, value}]` | `donut.yaml` |
| `pie` | Camembert | `[{label, value}]` | `pie.yaml` |
| `progress-ring` | Anneau radial single | `value` (0-100) | `progress-ring.yaml` |
| `multi-ring` | Activity rings (3 anneaux) | `[{label, value}]` (1–4) | `multi-ring.yaml` |
| `gauge` | Demi-cercle (jauge) | `value` (0-100) + `min`/`max` | `gauge.yaml` |
| `sparkline` | Mini ligne (sans axes) | `[number]` | `sparkline.yaml` |
| `kpi` | Big number + delta + sparkline | `{value, delta, trend: [number]}` | `kpi.yaml` |
| `heatmap-cal` | Heatmap calendaire (GitHub) | `[{date, value}]` | `heatmap-cal.yaml` |
| `bubble` | Bubble (x,y,r) | `[{x, y, r, label?}]` | `bubble.yaml` |
| `funnel` | Funnel (entonnoir) | `[{label, value}]` | `funnel.yaml` |
| `radar` | Radar / spider | `[{axis, value}]` (3–8 axes) | `radar.yaml` |
| `waterfall` | Waterfall (variations) | `[{label, value, type: 'positive'|'negative'|'total'}]` | `waterfall.yaml` |
| `step-progress` | Étapes de progression | `[{label, status: 'done'|'current'|'todo'}]` | `step-progress.yaml` |

## Workflow

### Étape 1 — Identifier l'intention

Parser la demande utilisateur :

```
{
  mode: 'single' | 'batch' | 'batch-showcase',
  type: <id du catalogue> | null,        // null si batch
  container: 'bare' | 'card' | 'kpi-card',
  data: <selon type> | null,             // null si batch
  count: <number>                         // pour batch
  fileKey: <Figma file key>,
  pageOrFrame: <node ID où poser> | null  // null = nouvelle page
}
```

Si données ambiguës pour un single → demander **brièvement** (1 question max) ou choisir des défauts plausibles.

### Étape 2 — Résoudre les tokens couleur (find-or-create)

> **Spec complète : [knowledge-base/local-tokens.md](./knowledge-base/local-tokens.md)**

Principe : **chaque couleur d'un chart est liée à une variable** (jamais hex direct), pour que l'utilisateur puisse tout re-styler depuis le panel Variables.

1. **Chercher dans le DS** (`search_design_system`) — patterns par rôle (cf. local-tokens.md table).
2. **Si trouvé** → `importVariableByKeyAsync` et utiliser la variable du DS.
3. **Si absent** → créer/utiliser une variable dans la collection locale `Charts Tokens` (créée à la volée si besoin) avec une valeur hex de fallback.
4. **Reporter** à l'utilisateur ce qui a été réutilisé du DS vs créé localement.

Jamais de doublon : si la collection `Charts Tokens` existe déjà avec un token de même nom, on le réutilise.

### Étape 3 — Build via `use_figma`

Stratégie 1 appel MCP par chart (ou 1 appel pour tout le batch si <20 charts).

#### Architecture standard d'une card-chart

```
Card Frame (VERTICAL auto-layout, FILL_CONTAINER, padding=24, gap=16)
  ├── Header (HORIZONTAL, space-between, HUG height)
  │   ├── Title group (VERTICAL, gap=4)
  │   │   ├── Title (text)
  │   │   └── Subtitle (text, optional)
  │   └── Action slot (HUG, optional)
  ├── Big value (text, optional — pour donut/kpi/progress)
  ├── Chart Frame (FILL_CONTAINER × FIXED height ou ratio)
  │   └── <SVG composé via vectors / rectangles>
  └── Legend (HORIZONTAL wrap, optional)
```

**Responsive — RÈGLE OBLIGATOIRE** :

1. Card : `layoutAlign='STRETCH'` si parent est auto-layout, sinon dimensions fixes.
2. Chart frame interne : `layoutMode='NONE'` + `layoutAlign='STRETCH'` + `layoutGrow=1` → remplit la place dans la card.
3. **TOUS les enfants du chart frame** (vectors, rectangles, ellipses, textes) doivent avoir `constraints = { horizontal: 'SCALE', vertical: 'SCALE' }` pour suivre le redimensionnement.
4. Appliquer en post-walk après le build :
   ```js
   const walk = (n) => {
     if (n !== body && 'constraints' in n) n.constraints = { horizontal: 'SCALE', vertical: 'SCALE' };
     if ('children' in n) for (const c of n.children) walk(c);
   };
   walk(body);
   ```

Sans cela, redimensionner la card laissera le chart figé en haut-gauche.

### Étape 4 — Données : injection ou aléatoire

> **Spec complète : [knowledge-base/data-injection.md](./knowledge-base/data-injection.md)**

Trois modes :

- **random** (défaut batch) : utilise les générateurs ci-dessous.
- **inline** : parse le prompt pour extraire `Jan=120, Feb=185...` ou `12, 18, 15, 24`.
- **JSON** : bloc ```json ... ``` avec schema strict par type (cf. data-injection.md).

Si l'utilisateur fournit `values` sans `labels` → générer auto. Si data fournie pour un mode `batch-showcase` → ignorer (random) et le signaler.

#### Génération de données aléatoires (mode random)

Utiliser des distributions plausibles :

| Type | Générateur |
|------|-----------|
| `line` / `area` / `sparkline` | random walk (drift léger), 7–30 points, range 100–1000 |
| `bar-v` / `bar-h` | poisson(λ=50), 5–8 catégories, labels Mon..Sun ou Q1..Q4 ou Jan..Dec |
| `donut` / `pie` | dirichlet(α=2) sur 3–5 segments, labels A/B/C ou Mobile/Desktop/Tablet |
| `progress-ring` / `gauge` | uniform(20, 95) |
| `multi-ring` | 3 valeurs uniform(40, 100) |
| `kpi` | value 1k–100k, delta -25%..+45%, trend random walk 12 pts |
| `heatmap-cal` | 365 jours, exponential(λ=0.3) |
| `funnel` | décroissant : 100, 75, 55, 35, 20 ± noise |
| `radar` | 5–6 axes uniform(30, 95) |

Toujours seed-able (`Math.random` dans le snippet `use_figma` — ok pour mock).

## Spécifications détaillées

Les specs par chart sont dans `knowledge-base/charts/<id>.yaml`. Format :

```yaml
id: line
title: Line Chart
size: { width: 432, height: 280 }     # défaut card content
data_shape: '[number] | { labels:[string], values:[number] }'
geometry:
  padding: { top: 20, right: 16, bottom: 28, left: 32 }
  axes:
    x: bottom labels, every Nth tick
    y: left, 4 ticks, formatter: 'k'
build:
  - polyline via vector network (one anchor per point)
  - area fill (optional) via closed path → gradient
  - dots optional (ellipse 8×8 at each anchor)
colors:
  line: chart/1
  area: chart/1 @ 20% opacity
  dots: chart/1
  grid: border/subtle
  axis: text/secondary
```

Voir [knowledge-base/charts/](./knowledge-base/charts/) pour les specs complètes.

## Pitfalls connus

| # | Symptôme | Cause | Fix |
|---|----------|-------|-----|
| P1 | Couleur en dur (hex) | `setBoundVariableForPaint` non appelé | Toujours résoudre les vars d'abord |
| P2 | Chart débordant la card | `FILL_CONTAINER` non set sur le chart frame | `layoutSizingHorizontal = 'FILL'` |
| P3 | Labels qui se chevauchent | Trop de catégories vs largeur | Skip 1/N labels ou rotation -45° |
| P4 | Donut center value mal centré | Pas en absolute position | Frame center via `constraints` ou auto-layout `CENTER` |
| P5 | Vector polyline distordue | Coordonnées en pixels relatives au mauvais parent | Toujours coords relatives au chart frame, vector posé en (0,0) |
| P6 | Stack mal aligné | Cumul = sum, mais % bar height oublié | Utiliser ratio = sum / max(sum) |
| P7 | Heatmap dates mal mappées | Semaine commence dimanche vs lundi | Documenter `weekStart` config (défaut Mon) |
| P8 | Radar axes irréguliers | angles non équidistants | `angle = -π/2 + i * 2π/N` |
| P9 | Chart figé après redimensionnement de la card | Constraints par défaut (LEFT_TOP) | Appliquer `constraints={horizontal:'SCALE',vertical:'SCALE'}` à tous les enfants du chart frame en post-walk |

## Knowledge Base

```
knowledge-base/
├── charts/             # 1 YAML par type de chart
│   ├── line.yaml
│   ├── bar-v.yaml
│   ├── donut.yaml
│   └── ...
├── colors.md           # Mapping rôle ↔ patterns de noms DS
├── local-tokens.md     # Stratégie find-or-create + palette par défaut
├── data-injection.md   # Format des données utilisateur (inline / JSON)
├── data-generators.md  # Générateurs aléatoires par type (mode random)
├── containers.md       # Spec des 3 modes (bare / card / kpi-card)
├── visual-style.md     # Style "graphique" : big numbers, pills, anneaux décoratifs
└── geometry.md         # Helpers : scale, polyline, arc, donutSlice
```

## Style visuel

> **Spec complète : [knowledge-base/visual-style.md](./knowledge-base/visual-style.md)**

Style inspiré du Smart Charts Kit, plus aéré et graphique que les premiers builds :
- Cards 432×440 (ou plus) au lieu de 320×320.
- Big number dominant (36-56pt) en tête de chart.
- Pills pour deltas, légendes, annotations.
- Anneaux décoratifs concentriques en bg pour radial.
- Aires en gradient vertical (couleur 25% → 0%).
- Bars épaisses arrondies (radius = width/2 → pill verticale).
- Bouton "Details" en bas (pill rempli, optionnel).

## Référence visuelle

Smart Charts Kit (Figma Community) — utilisé comme north-star UI :
`https://www.figma.com/design/unAQSfuwGyYnv4wQyNTO8h/31-Free-Smart--Charts-Kit`

Pour reproduire un chart précis du kit, appeler `get_design_context` sur le node ID correspondant et adapter au DS cible.
