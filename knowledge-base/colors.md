# Color Resolution Strategy

Toujours résoudre les couleurs **via variables Figma** — jamais de hex en dur dans le rendu final.

## Étape 1 — Chercher la collection chart

```js
// via search_design_system
search_design_system({ query: 'chart', includeVariables: true })
search_design_system({ query: 'data viz', includeVariables: true })
```

Hiérarchie de recherche (dans l'ordre) :

1. **Collection dédiée** : `chart/1`..`chart/N`, `chart/categorical/*`
2. **Sémantique** : `chart/positive`, `chart/negative`, `chart/neutral`, `chart/info`
3. **Brand** : `brand/primary`, `brand/secondary`, `brand/tertiary`
4. **Sémantique générale** : `semantic/success`, `semantic/danger`, `semantic/warning`, `semantic/info`

## Étape 2 — Variables UI

Toujours utiliser pour les éléments structurels :

| Rôle | Variable cible | Fallback |
|------|---------------|----------|
| Card surface | `surface/default` | `bg/primary` |
| Card border | `border/subtle` | `border/default` |
| Grid lines | `border/subtle` | — |
| Axis labels | `text/secondary` | `text/muted` |
| Title | `text/primary` | `text/default` |
| Subtitle | `text/secondary` | — |
| Big value | `text/primary` | — |
| Tooltip bg | `surface/inverse` | `bg/inverse` |

## Étape 3 — Lier au paint

```js
const paint = figma.util.solidPaint('#000000'); // placeholder
const bound = figma.variables.setBoundVariableForPaint(paint, 'color', variable);
node.fills = [bound];
```

Pour les **opacités** (ex: aire sous courbe à 20%) :

```js
const fill = { type: 'SOLID', color: { r: 0, g: 0, b: 0 }, opacity: 0.2 };
const bound = figma.variables.setBoundVariableForPaint(fill, 'color', variable);
node.fills = [bound];
```

L'opacity est respectée même quand color est lié.

## Étape 4 — Gradients (area charts)

Gradient vertical chart-color → transparent :

```js
const gradient = {
  type: 'GRADIENT_LINEAR',
  gradientTransform: [[0, 1, 0], [-1, 0, 1]], // top to bottom
  gradientStops: [
    { position: 0, color: { r: 0, g: 0, b: 0, a: 0.25 } },
    { position: 1, color: { r: 0, g: 0, b: 0, a: 0    } },
  ],
};
// Bind first stop's color to chart variable
gradient.gradientStops[0] = figma.variables.setBoundVariableForPaint(
  gradient.gradientStops[0], 'color', chartVar
);
```

## Étape 5 — Mode batch

Pour un batch, **assigner une variable différente par série** en cyclant `chart/1`..`chart/N`. Si N catégories > N variables, recycler avec opacity dégradée.

## Logging

Si une variable cible manque, logger dans la réponse :

> ⚠️ Variable `chart/3` introuvable dans le DS — fallback sur `brand/secondary`.

L'utilisateur peut alors ajouter la variable au DS pour les prochains builds.
