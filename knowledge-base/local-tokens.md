# Stratégie tokens — Find or Create

Pour que les charts soient **éditables centralement** (changer une couleur dans le panel Variables et tout le dashboard se met à jour), **chaque couleur d'un chart doit être liée à une variable**, jamais en hex direct.

## Stratégie "find or create"

À chaque génération, le skill :

1. **Cherche** dans le DS du fichier (via `search_design_system`) les variables candidates.
2. **Si trouvées** → utilise les variables du DS (priorité absolue, jamais de doublon).
3. **Si manquantes** → crée une collection locale `Charts Tokens` avec les variables manquantes, puis les utilise.
4. **Loggue** dans la réponse à l'utilisateur ce qui a été trouvé vs créé.

## Mapping des rôles attendus

Chaque chart consomme un **rôle**, pas une variable précise. Le résolveur essaye dans cet ordre :

| Rôle | Patterns recherchés dans le DS | Token local créé si absent |
|------|--------------------------------|----------------------------|
| `series.1` à `series.8` | `chart/1..8`, `chart/series/*`, `chart/legends/*`, `data/*`, `viz/*` | `chart/series/1..8` |
| `positive` | `chart/positive`, `semantic/success`, `feedback/success`, `*/success` | `chart/positive` |
| `negative` | `chart/negative`, `semantic/danger`, `semantic/error`, `*/error` | `chart/negative` |
| `warning` | `chart/warning`, `semantic/warning`, `*/warning` | `chart/warning` |
| `neutral` | `chart/neutral`, `text/secondary`, `*/muted` | `chart/neutral` |
| `surface` | `surface/default`, `surface/primary`, `background/card`, `*/card` | `chart/surface` |
| `border` | `border/subtle`, `border/default`, `*/divider` | `chart/border` |
| `text.primary` | `text/primary`, `text/default/default`, `content/primary` | `chart/text/primary` |
| `text.secondary` | `text/secondary`, `text/muted`, `content/secondary` | `chart/text/secondary` |
| `grid` | `chart/grid`, `border/subtle` | `chart/grid` |
| `track` | `chart/track`, `surface/subtle`, `*/disabled-bg` | `chart/track` |

## Création de la collection locale

Une seule fois par fichier — vérifier d'abord si elle existe :

```js
async function getOrCreateChartCollection() {
  const collections = await figma.variables.getLocalVariableCollectionsAsync();
  let coll = collections.find(c => c.name === 'Charts Tokens');
  if (!coll) {
    coll = figma.variables.createVariableCollection('Charts Tokens');
  }
  return coll;
}
```

## Création d'une variable locale

```js
async function getOrCreateColorVar(coll, name, hexFallback) {
  const all = await figma.variables.getLocalVariablesAsync('COLOR');
  let v = all.find(x => x.variableCollectionId === coll.id && x.name === name);
  if (!v) {
    v = figma.variables.createVariable(name, coll, 'COLOR');
    const modeId = coll.modes[0].modeId;
    v.setValueForMode(modeId, hexToRgb(hexFallback));
  }
  return v;
}
```

## Palette par défaut (utilisée si création locale)

Inspirée du Smart Charts Kit (ton accentué, bonne lisibilité) :

| Token | Hex |
|-------|-----|
| `chart/series/1` | `#5B6BE7` (indigo) |
| `chart/series/2` | `#F0AC4C` (orange) |
| `chart/series/3` | `#5BC59A` (green) |
| `chart/series/4` | `#E26AA1` (pink) |
| `chart/series/5` | `#7E5BE7` (purple) |
| `chart/series/6` | `#4CCBE0` (cyan) |
| `chart/series/7` | `#E7E15B` (yellow) |
| `chart/series/8` | `#5B8FE7` (sky) |
| `chart/positive` | `#5BC59A` |
| `chart/negative` | `#E25B5B` |
| `chart/warning` | `#F0AC4C` |
| `chart/neutral` | `#9AA3B2` |
| `chart/surface` | `#FFFFFF` |
| `chart/border` | `#E5E7EB` |
| `chart/text/primary` | `#0F172A` |
| `chart/text/secondary` | `#64748B` |
| `chart/grid` | `#EFF1F4` |
| `chart/track` | `#EEF0F3` |

## Algorithme de résolution

```js
async function resolveRoles(fileKey) {
  const found = {};   // role -> imported DS variable
  const created = {}; // role -> locally created variable
  const coll = await getOrCreateChartCollection();
  const localVars = await figma.variables.getLocalVariablesAsync('COLOR');
  const localByName = Object.fromEntries(localVars.map(v => [v.name, v]));

  for (const role of ROLES) {
    // 1. Search DS for any of the patterns of this role
    const dsHit = await searchDsForRole(role); // returns { libraryKey, key } or null
    if (dsHit) {
      found[role] = await figma.variables.importVariableByKeyAsync(dsHit.key);
      continue;
    }
    // 2. Use existing local
    const localName = ROLE_LOCAL_NAME[role];
    if (localByName[localName]) {
      created[role] = localByName[localName];
      continue;
    }
    // 3. Create local
    const v = figma.variables.createVariable(localName, coll, 'COLOR');
    v.setValueForMode(coll.modes[0].modeId, hexToRgb(DEFAULT_PALETTE[role]));
    created[role] = v;
  }
  return { found, created };
}
```

## Reporting

Après build, retourner à l'utilisateur :

```
✓ DS variables réutilisées (5) :
  - series.1 → color/chart/legends/primary (Semji Core)
  - series.2 → color/chart/legends/green (Semji Core)
  - positive → color/feedback/success (Semji Core)
  ...
+ Tokens locaux créés (4) dans collection « Charts Tokens » :
  - chart/surface = #FFFFFF
  - chart/border = #E5E7EB
  - chart/text/primary = #0F172A
  - chart/grid = #EFF1F4
```

Ainsi l'utilisateur sait quoi modifier où pour ré-éditer.
