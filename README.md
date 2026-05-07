# ds-charts-skill

> Génère des graphiques de dashboard dans Figma via MCP — line, bar, donut, gauge, sparkline, heatmap, radar, funnel, KPI, et plus. Couleurs liées aux variables du Design System, layout responsive (auto-layout), 2 modes : data fournie ou batch aléatoire.

## Installation

Place ce dossier dans `~/.copilot/skills/` (ou ton dossier de skills VS Code).

Le skill s'auto-active dès que l'utilisateur demande un graphique pour Figma.

## Prérequis

- Figma MCP server connecté (voir [README skills root](../README.md))
- Un fichier Figma cible (avec idéalement un Design System avec variables `chart/*`, `surface/*`, `text/*`, `border/*`)

## Usage

### Chart unitaire avec données

```
"Crée un line chart avec [10, 25, 18, 40, 32, 55, 48] dans une card 'Revenue Q4'"
"Donut avec Mobile 60%, Desktop 30%, Tablet 10%"
"Progress ring à 75% — 'Goal'"
"KPI card : Revenue $42,580, +12.5%, trend [...]"
```

### Batch aléatoire

```
"Génère 12 charts d'exemple pour mon dashboard"
"Un de chaque type — showcase complet"
"Remplis cette page de KPI cards (4 colonnes)"
```

### Style visuel

```
"... style data"          → compact 380px, fond blanc, bordure fine, no shadow — pour dashboards denses
"... compact"             → idem (alias auto-détecté)
"... style graphical"     → card polished 432px+, gradients, ombres, big number — défaut
```

### Container

```
"... en mode bare (sans card)"
"... en kpi-card"
"... dans une card avec footer 'View details'"
```

## Catalogue (20 types)

**Line family** — line, multi-line, area, sparkline
**Bar family** — bar-v, bar-h, bar-grouped, bar-stacked
**Radial** — donut, pie, progress-ring, multi-ring, gauge
**Specialized** — kpi, heatmap-cal, bubble, funnel, radar, waterfall, step-progress

## Structure

```
ds-charts-skill/
├── SKILL.md                       # entry point — routing + workflow
├── README.md                      # ce fichier
└── knowledge-base/
    ├── colors.md                  # stratégie variables couleur
    ├── containers.md              # bare / card / kpi-card
    ├── data-generators.md         # JS snippets pour données aléatoires
    ├── geometry.md                # helpers : scale, polyline, donut, etc.
    └── charts/
        ├── line-family.md         # line, multi-line, area, sparkline
        ├── bar-family.md          # bar-v, bar-h, grouped, stacked
        ├── radial-family.md       # donut, pie, progress, multi-ring, gauge
        └── specialized.md         # kpi, heatmap, bubble, funnel, radar, waterfall, step
```

## Limites connues

- Les vectors Figma ne reflow pas automatiquement quand le parent change de taille → re-générer le chart
- Maximum recommandé : 20 charts par appel `use_figma` (au-delà, splitter en plusieurs appels)
- Heatmap calendaire : 365 cells max recommandé
