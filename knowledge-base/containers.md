# Containers

3 modes d'enveloppe pour un chart.

## Mode `bare`

Frame transparent contenant uniquement la géométrie du chart.

```
Chart Frame (auto-layout NONE, fills=[], FILL_CONTAINER × FIXED height)
  └── <vectors / rectangles>
```

Cas d'usage :
- Insertion dans un layout existant (l'utilisateur a déjà sa card)
- Composition libre

Tailles défaut : `width=auto (fill)`, `height=200`

## Mode `card` (défaut)

Card DS standard avec header, body chart, footer optionnel.

```
Card (VERTICAL, FILL × HUG, padding=24, gap=16, cornerRadius=12, fills=surface/default, stroke=border/subtle 1px)
  ├── Header (HORIZONTAL, FILL × HUG, space-between, gap=16)
  │   ├── Titles (VERTICAL, HUG × HUG, gap=4)
  │   │   ├── Title (text 16/600, text/primary)
  │   │   └── Subtitle (text 13/400, text/secondary, optional)
  │   └── Action (HUG, optional — badge / button / menu)
  ├── BigValue (text 32/700, text/primary, optional)
  │   └── Delta inline (text 14/500, semantic/success ou /danger)
  ├── Chart (FILL × FIXED height=200)
  └── Legend (HORIZONTAL wrap, FILL × HUG, gap=12, optional)
      └── LegendItem × N (HORIZONTAL, gap=6)
          ├── Dot (8×8 ellipse, color = chart/i)
          └── Label (text 12/400, text/secondary)
```

Tailles défaut : `width=432`, `height=auto (HUG)`.

> ⚠️ **Ne jamais mettre de hauteur fixe sur la card** — utiliser `layoutSizingVertical='HUG'` pour que la card s'adapte à son contenu. Seul le frame chart interne a une hauteur fixe. Une hauteur fixe sur la card provoque la coupure du contenu.

### Header — variantes

| Variante | Contenu |
|----------|---------|
| `title-only` | Title seul |
| `title-subtitle` | Title + Subtitle |
| `title-action` | Title + Action (badge période, menu, etc.) |
| `title-subtitle-action` | Tous |

### Footer (optional)

```
Footer (HORIZONTAL, FILL × HUG, space-between)
  ├── Caption (text 12/400, text/secondary)
  └── Link (text 12/500, brand/primary, "View details →")
```

## Mode `kpi-card`

Big number dominant + delta + sparkline en bas.

```
KPI Card (VERTICAL, FILL × HUG, padding=20, gap=12, cornerRadius=12, fills=surface/default)
  ├── Label (text 13/500, text/secondary, uppercase)
  ├── Row (HORIZONTAL, FILL × HUG, align=baseline, gap=8)
  │   ├── Value (text 32/700, text/primary)
  │   └── Delta (HORIZONTAL, HUG, gap=2)
  │       ├── Arrow icon (12×12, semantic/success ou /danger)
  │       └── DeltaText (text 13/500, semantic/success ou /danger)
  └── Sparkline (FILL × FIXED height=48, no axes, no labels)
```

Tailles défaut : `width=240`, `height=auto`. Idéal en grid 4 colonnes.

## Responsive — RÈGLE OBLIGATOIRE

Pour que les charts se redimensionnent quand l'utilisateur étire la card après coup :

1. **Card** : `layoutAlign='STRETCH'` si parent est auto-layout, sinon dimensions fixes raisonnables.
2. **Chart frame (body)** : `layoutMode='NONE'` + `layoutAlign='STRETCH'` + `layoutGrow=1` → remplit la place restante de la card.
3. **TOUS les enfants du chart frame** (vectors, rectangles, ellipses, textes) **doivent recevoir** :
   ```js
   node.constraints = { horizontal: 'SCALE', vertical: 'SCALE' };
   ```
   À appliquer après création de chaque shape ou en post-walk récursif :
   ```js
   const walk = (n) => {
     if (n !== body && 'constraints' in n) {
       n.constraints = { horizontal: 'SCALE', vertical: 'SCALE' };
     }
     if ('children' in n) for (const c of n.children) walk(c);
   };
   walk(body);
   ```
4. Pour un layout dashboard : parent Grid auto-layout `HORIZONTAL` avec `layoutWrap='WRAP'`, cards en largeur fixe (320–432) → wrap automatique selon la largeur dispo.

**Sans contraintes SCALE, le redimensionnement de la card laisse le chart figé en haut-gauche → ne pas oublier.**
