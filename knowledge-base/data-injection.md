# Data injection — Format & convention

Le skill accepte des **données dynamiques** passées par l'utilisateur dans son prompt OU des **données aléatoires** générées via `data-generators.md`.

## 3 modes

### Mode 1 — random (par défaut)

> "génère un line chart"
> "fais-moi un dashboard avec tous les types"

Aucune donnée fournie → utiliser les générateurs. Idéal pour showcase / template.

### Mode 2 — single chart avec data inline

> "génère un bar chart avec ces ventes : Jan=120, Feb=185, Mar=140, Apr=210, May=190"
> "donut: Direct 45%, Search 30%, Social 15%, Other 10%"
> "kpi: revenue $48,200 +12.4% sparkline=12,18,15,24,22,28,32"

Le skill **parse le prompt** pour extraire :
- Labels (séparés par `,` ou `=`)
- Valeurs (numériques)
- Title (si entre guillemets)
- Delta (si `+X%` ou `-X%`)

### Mode 3 — JSON structuré (multi-chart ou data complexe)

L'utilisateur peut passer un bloc JSON. Le skill lit `data` et l'attribue par index ou par `id` :

```json
{
  "title": "Q1 Performance",
  "charts": [
    {
      "type": "kpi",
      "title": "Revenue",
      "value": "$48.2k",
      "delta": "+12.4%",
      "sparkline": [12, 18, 15, 24, 22, 28, 32]
    },
    {
      "type": "line",
      "title": "Daily visits",
      "labels": ["Mon","Tue","Wed","Thu","Fri","Sat","Sun"],
      "series": [
        {"name": "This week", "values": [120, 145, 132, 178, 195, 210, 188]},
        {"name": "Last week", "values": [110, 130, 125, 150, 165, 180, 170]}
      ]
    },
    {
      "type": "donut",
      "title": "Traffic sources",
      "segments": [
        {"label": "Direct", "value": 45},
        {"label": "Search", "value": 30},
        {"label": "Social", "value": 15},
        {"label": "Other", "value": 10}
      ]
    }
  ]
}
```

## Schémas par type de chart

| Type | Schema |
|------|--------|
| `kpi` | `{title, value, delta?, deltaPositive?, sparkline?: number[]}` |
| `line` / `area` | `{title, labels?: string[], series: [{name, values: number[]}]}` |
| `multi-line` | identique à `line` mais ≥ 2 séries |
| `bar-v` / `bar-h` | `{title, labels: string[], values: number[]}` |
| `bar-grouped` | `{title, labels: string[], series: [{name, values: number[]}]}` |
| `bar-stacked` | identique à grouped |
| `donut` / `pie` | `{title, segments: [{label, value}]}` |
| `progress-ring` | `{title, value: 0-100, label?}` |
| `multi-ring` | `{title, rings: [{label, value: 0-100}]}` |
| `gauge` | `{title, value: 0-100, min?, max?}` |
| `sparkline` | `{values: number[], delta?, deltaPositive?}` |
| `heatmap-cal` | `{title, weeks: number, values: number[7*weeks]}` |
| `bubble` | `{title, points: [{x, y, r, label?}]}` |
| `funnel` | `{title, stages: [{label, value}]}` |
| `radar` | `{title, axes: string[], series: [{name, values: number[]}]}` |
| `waterfall` | `{title, items: [{label, value, type?: 'in'|'out'|'total'}]}` |
| `step-progress` | `{title, steps: [{label, status: 'done'|'current'|'pending'}]}` |

## Pattern de parsing dans le skill

```js
function parseUserData(prompt) {
  // 1. JSON block?
  const jsonMatch = prompt.match(/```json\s*([\s\S]*?)\s*```/);
  if (jsonMatch) return JSON.parse(jsonMatch[1]);

  // 2. Inline pairs?  "Jan=120, Feb=185"
  const pairs = [...prompt.matchAll(/([A-Za-zÀ-ÿ][\w\s]*)\s*[=:]\s*([\d.]+)/g)];
  if (pairs.length >= 2) {
    return { labels: pairs.map(p => p[1].trim()), values: pairs.map(p => +p[2]) };
  }

  // 3. Inline number list?  "12, 18, 15, 24, 22"
  const nums = [...prompt.matchAll(/-?\d+(\.\d+)?/g)].map(m => +m[0]);
  if (nums.length >= 3) return { values: nums };

  return null; // → fallback to random
}
```

## Règles UX

- Si l'utilisateur fournit `values` mais pas `labels` → générer auto (J1, J2, ... ou Mon, Tue, ...).
- Si une valeur dépasse le `max` (donut > 100%) → log warning + normalize.
- Toujours **conserver les labels exacts** fournis (pas de transformation).
- Si l'utilisateur demande "showcase" / "tous les types" → ignorer ses datas pour ce build et utiliser random sur les 20 types.

## Reporting à l'utilisateur

Après build :

```
✓ Data
  - Mode: user-provided (parsed from prompt)
  - Type: bar-v
  - Series: 1 × 5 points
  - Labels: Jan, Feb, Mar, Apr, May
```

Ou :

```
✓ Data
  - Mode: random (no data provided)
  - Generators used: monthlyTrend, weeklyDistribution, ...
```
