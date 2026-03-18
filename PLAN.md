# Plan — AoE2 Build Order Web App

Paste the raw HTML of any aoecompanion.com build order page and ask me to generate a new build order page following this spec.

---

## File structure

```
ageOf2BO/
├── index.html          ← landing page listing all available build orders
├── archer-rush.html    ← Archer Rush
├── [strategy-slug].html  ← one file per build order, added on request
└── PLAN.md             ← this file
```

### Rules for the landing page (`index.html`)
- Lists every build order as a clickable card.
- Each card shows: strategy name, short description, tags (Rush / Boom / Féodal / Châteaux / etc.), and the 3 key stats (Vills. féodaux, Temps féodal, Premier [unit]).
- Cards link to `[strategy-slug].html`.
- A filter bar at the top lets the user filter by tag (Tous / Rush / Boom / Féodal / Châteaux). Filtering is done client-side via `data-tags` attribute on each card.
- **When a new build order is added, `index.html` must be updated** with the new card (name, description, tags, stats, slug).
- Same light theme and CSS variables as the build order pages (see §5).

---

## Hard Constraints (from previous requests)

### 1 · Output format
- Each build order is a single self-contained static HTML file (e.g. `archer-rush.html`), no external CSS or JS files.
- All images are loaded from remote URLs (AoE Companion S3); nothing is embedded.
- A **Back** link (`← Ordres de jeu`) at the top of every build order page links back to `index.html`.

---

### 2 · Language
- **UI chrome** (labels, buttons, hints, age names, building names, resource names, unit names): **French**, using the official names from the French AoE2 wiki (`ageofempires.fandom.com/fr`).
- **Technology names** (researched upgrades such as Loom, Fletching, Double-Bit Axe, Wheelbarrow, Horse Collar, Gold Mining, etc.): **English**, exactly as written in the game / aoecompanion.com.
- **Build order strategy names** (e.g. Archer Rush, Men-at-Arms + Archers, Fast Castle into Knights, Scout Rush): **never translated**. Use the exact English name from aoecompanion.com as both the `<h1>` and the `<title>` tag. The rest of the page content remains in French.
- Key French glossary to reuse:

| English | French |
|---|---|
| Dark Age | Âge sombre |
| Feudal Age | Âge féodal |
| Castle Age | Âge des châteaux |
| Imperial Age | Âge impérial |
| Town Center | Forum |
| House | Maison |
| Lumber Camp | Camp de bûcherons |
| Mining Camp | Camp minier |
| Mill | Moulin |
| Barracks | Casernes |
| Archery Range | Camp de tir à l'arc |
| Blacksmith | Forge |
| Farm | Ferme(s) |
| Stable | Écurie |
| Villager | Villageois |
| Sheep | Moutons |
| Boar | Sanglier |
| Berries | Baies |
| Wood | Bois |
| Gold | Or |
| Food | Nourriture |
| Archer | Archer |

---

### 3 · Icons
- Use images from the AoE Companion S3 bucket (`https://aoecompanion.s3.eu-central-1.amazonaws.com/`) for all action icons and resource icons.
- Icon filenames are found in the source HTML of aoecompanion.com build order pages (look for `<img>` tags pointing to that bucket).
- Every action row must display a 30×30px icon on the left.

---

### 4 · Resource distribution row
- Each build order step must include a row of small resource chips below the actions showing **how many villagers are on each resource after that step**.
- Chips use a 16×16px icon (from the S3 bucket) + label + count.
- **Sheep + Boar must be merged** into a single "Moutons/Sanglier N" chip whenever both are non-zero.
- Villager counts must be derived by tracking cumulative assignments from pop 3, anchored to the two confirmed resource splits published by aoecompanion.com:
  - At Feudal Age entry (from the build order page)
  - At Castle Age entry (from the build order page)

---

### 5 · Visual design — light theme
CSS variables to use exactly (both on `index.html` and all build order pages):
```css
--bg:      #f7f5f2;
--surface: #eeebe6;
--line:    #ddd9d2;
--muted:   #9a9188;
--body:    #4a4540;
--head:    #1a1714;
--gold:    #8a6a10;
--gold-dim:#b8952a;
```
Resource fallback square colours:
```css
.rs-sheep  { background: #c0903a; }
.rs-boar   { background: #8a5828; }
.rs-berry  { background: #a04868; }
.rs-wood   { background: #7a6038; }
.rs-gold   { background: #c8a010; }
.rs-farm   { background: #5a8840; }
```

---

### 6 · Sections to include (and only these)
1. **Back link** — `← Ordres de jeu` linking to `index.html`.
2. **Header** — eyebrow (`Age of Empires II · Édition Définitive`), strategy name as `<h1>`, tagline (`Ordre de jeu · 1v1 Arabie`), DLC note if applicable.
3. **Stats row** — 3 columns: Vills. féodaux / Temps féodal / Premiers [unit] (values taken from the build order).
4. **Controls** — `X / Y étapes terminées` counter + `Réinitialiser` button.
5. **Age label + step list** for each age (Âge sombre, Âge féodal, …).
6. **Transition banners** between ages — title, resource costs with icons, transition note.
7. **Split bars** — proportional coloured bar + icon legend showing villager split at each age-up.
8. **Footer** — source credit + timing disclaimer.

### Sections to exclude
- Difficulty rating
- Best Civilisations
- Execution Tips
- Any other editorial content from the page

---

### 7 · Interactivity
- Clicking any step toggles it `.done` (opacity 0.25, action name struck through).
- A fixed top progress bar fills as steps are completed.
- `Réinitialiser` clears all done states.
- Step population column shows the pop count in gold (`var(--gold)`).

---

### 8 · Key action highlight
Every build order has one defining action that the player must not miss — the thing that makes the strategy unique. This action must be highlighted as a red badge inside its `then` or `desc` text, using this exact inline style:

```html
<strong style='display:inline-block;background:rgba(192,64,64,.12);color:#c04040;border:1px solid rgba(192,64,64,.3);border-radius:4px;padding:1px 7px;font-size:12px;font-weight:700;letter-spacing:.3px'>Text here</strong>
```

Examples of what to highlight:
- MAA + Archers → **"Former 3 Miliciens"** (pop 19 Barracks `then` note)
- Archer Rush → **"Formez 6–8 Archers"** (pop 21 Feudal Archery Range `then` note)
- A boom → the farm or TC production trigger

Identify the defining action from the build order content and apply this badge to it. Only one highlight per build order.

---

### 9 · Age-up step layout
The pop-21 (or equivalent) step that triggers the age advance must be split by a dashed gold divider (`<hr class="age-up-divider">`) into:
- **Before the divider**: actions to do immediately (queue new villager, research Loom, click age-up).
- **After the divider**: reassignments to do *while* aging up (redirect villagers to wood/gold/barracks).

---

### 10 · DLC / version note
If the build order is from a post-February 2026 version of the game, include:
> ↑ Mis à jour pour le DLC The Last Chieftains, fév. 2026

---

### 11 · Print mode (landscape, single page)
Every build order page must include a `@media print` block that produces a clean, single-page reference card when printed in **A4 landscape**:

- `@page { size: A4 landscape; margin: 0.6cm 1cm; }`
- **Hidden** in print: progress bar, controls row, back link, vill hints, DLC note, footer, transition banners, split bars, split legends, `.action-desc`, `.action-from`.
- **Shown** in print: header (compact flex row), stats row, age labels, all step lists, key-action badge.
- Step lists use **3 CSS columns** (`columns: 3; column-gap: 8px`) so dark-age steps flow across the page.
- Each `.step` overridden to `display: flex` (replacing the default grid) with tightly reduced padding and font sizes (~7–8.5 px).
- All resource chips and action icons scaled down to fit (icons 13×13 px, res-img 8×8 px).
- The key-action badge (`<strong>`) must remain visible, with slightly smaller font/padding.
- Use `-webkit-print-color-adjust: exact` to preserve colours.

---

## How to request a new build order

1. Go to `aoecompanion.com/build-guides/<strategy-name>`.
2. Open DevTools → Elements, select the full page `<html>` and copy the outer HTML.
3. Paste it in the chat with a message like:

> "Ajoute le build order pour [strategy name] avec ce contenu : [paste HTML]"

I will:
1. Extract the build order steps, icons, resource splits, transition details, and stats from the pasted HTML.
2. Create `[strategy-slug].html` following this spec.
3. Add a new card to `index.html` (name, description, tags, stats, link).
