# 📐 Guide des Dimensions et Résolutions pour les Animations Web

> **Version :** 1.0  
> **Date :** 2026-05-07  
> **Objectif :** Définir les tailles d’images et les résolutions optimales pour chaque usage animé ou statique, en conciliant qualité visuelle, performance et contraintes techniques.

---

## 1. Introduction

Choisir la mauvaise dimension pour une image animée (ou statique) entraîne :

-   Un **gaspillage de bande passante** si l’image est trop grande.
-   Une **perte de netteté** si elle est trop petite et agrandie par le navigateur.
-   Des **saccades ou une consommation excessive de mémoire** sur les appareils modestes.

Ce guide pose les bases professionnelles pour choisir la résolution adaptée à chaque contexte : icône, illustration, sprite sheet, fond animé, etc. Il s’applique à toutes les images du web (PNG, WebP, GIF, APNG, JPEG) et aux techniques d’animation classiques (CSS, sprite sheets, canvas).

---

## 2. Concepts clés

| Terme | Définition |
| :--- | :--- |
| **Pixel CSS (px)** | Unité logique utilisée en CSS. Indépendante de la densité de l’écran. |
| **Pixel physique** | Point lumineux réel de l’écran. |
| **devicePixelRatio (DPR)** | Rapport entre pixels physiques et pixels CSS. Ex. `DPR = 2` signifie qu’un carré de 1 px CSS occupe 2×2 pixels physiques. |
| **Résolution native** | Taille en pixels physiques de l’image. Une image de 200×200 px affichée en 100×100 px CSS sur un écran `DPR 2` utilise exactement sa résolution native. |
| **Résolution effective** | Taille d’affichage dans le navigateur (largeur × hauteur en CSS). |
| **Image haute densité** | Image produite à une résolution supérieure (ex. ×2 ou ×3) pour conserver sa netteté sur les écrans Retina ou haute résolution. |

---

## 3. Règles d’or

1.  **Toujours définir la taille d’affichage en CSS** (largeur/hauteur) et fournir une image dont la résolution native couvre le pire des cas de `devicePixelRatio` attendu.
2.  **Ne pas dépasser la taille d’affichage réelle × DPR max.** Sur le web, `DPR 3` est un plafond raisonnable (smartphones très haute densité).
3.  **Une image plus grande que nécessaire est inutile** : elle alourdit la page sans gain visuel.
4.  **Pour les animations, chaque kilo‑octet compte** : une sprite sheet de 500 Ko peut convenir, mais 2 Mo est excessif.
5.  **Utiliser le responsive loading** (`srcset`, `<picture>`, `media queries`) quand l’image change de taille selon l’écran.
6.  **Respecter les limites techniques** : pas d’image de plus de 16 384 px de largeur ou hauteur (navigateurs Chromium), éviter les images de plus de 50 Mpx pour la mémoire GPU.

---

## 4. Dimensions recommandées par cas d’usage

### 4.1. Icônes animées (favicon, chargeur, icône de menu)

-   **Taille d’affichage courante** : 16×16 px (favicon), 24×24 px, 32×32 px, 64×64 px.
-   **Résolution d’image** : prévoyez au moins **×2** pour le DPR. Par exemple, une icône de 32×32 px CSS → image en 64×64 px.
-   **Sprite sheet** : possible mais rare. Si utilisée, gardez la hauteur de l’image à 64 px max et le nombre de frames faible (≤ 30) pour un poids très contenu.
-   **Formats** : PNG‑24 (avec alpha), GIF pour compatibilité maximale, WebP animé si moderne. GIF limité à 256 couleurs mais souvent suffisant pour des icônes simples.

### 4.2. Illustrations, objets 3D en fallback, logos tournants

-   **Taille d’affichage** : généralement entre 150 px et 400 px de large (voire 600 px sur desktop pour un visuel d’accompagnement).
-   **Résolution d’image** : 
    -   Si l’image est statique ou animée simplement : prévoyez **×2** (ex. 300 px CSS → 600 px natif).
    -   Si elle doit rester nette sur des écrans DPR 3 : 900 px pour 300 px CSS.
    -   **Compromis acceptable** : visez 1,5× à 2×. L’œil distingue difficilement au‑delà.
-   **Cas des sprite sheets** : 
    -   La résolution de chaque frame doit être celle de l’illustration prévue (ex. 200 px). La largeur totale = `nb frames × largeur frame`.
    -   Pour 30 frames de 200 px → sprite de 6000 px de large. C’est raisonnable (< 16 384 px).
    -   Si la sprite devient trop large (> 10 000 px), réduire la résolution de chaque frame (150 px au lieu de 200 px) ou diminuer le nombre de frames (passer de 30 FPS à 24 FPS).
-   **Poids cible** : 200–400 Ko pour une illustration moyenne. Acceptable jusqu’à 800 Ko si l’image est centrale. Au‑delà, préférez une vidéo ou un APNG.

### 4.3. Hero section / fond animé

-   **Taille d’affichage** : pleine largeur, souvent `100vw`, hauteur variable. Exemple courant : 1920×1080 px en plein écran.
-   **Résolution d’image** :
    -   Pour une image fixe ou une vidéo : **au moins 1920 px de large** (voire 2560 px pour le DPR des grands écrans).
    -   Pour une animation via sprite sheet : une telle résolution est **impossible** en sprite (1920 px par frame × 30 frames = 57 600 px de large). On préférera donc **une vidéo (mp4/webm)** ou un canvas 3D.
    -   Si un fond animé doit être en sprite sheet (exemple : motif répétitif), réduisez drastiquement la taille de la tuile (100–300 px).
-   **Recommandation** : évitez les sprite sheets pour des fonds plein écran. Utilisez une vidéo courte avec `autoplay loop muted playsinline`, ou un dégradé CSS animé.

### 4.4. Sprite sheets (planches d’animation)

Cette section détaille le dimensionnement spécifique des sprite sheets.

#### a) Largeur d’une frame
-   Correspond à la taille d’affichage CSS prévue, multipliée par le DPR cible.
-   Exemple : objet affiché en 200 px CSS sur un écran DPR 2 → frame de 400 px. Mais cela augmente énormément la largeur totale. La plupart des projets utilisent des frames en pixels CSS (1×), acceptant une légère perte de netteté sur écrans Retina, car le gain de poids est considérable.
-   **Norme professionnelle** : pour une sprite sheet, on produit souvent des frames en **1×** (taille d’affichage CSS), et on applique une astuce CSS `image-rendering: auto` (ou `pixelated` pour un style pixel art) pour un rendu acceptable.

Si vous tenez à la haute densité, créez deux sprite sheets (1× et 2×) et utilisez des `media queries` de résolution pour basculer entre les deux.

#### b) Nombre de frames et largeur totale
-   Limite navigateur : **16 384 px** de largeur (Chromium) ou 8192 px (anciens Safari).
-   Pour rester en dessous de 10 000 px (sécurité) : nb max de frames = 10 000 / largeur d’une frame.
-   Si vous devez dépasser, optez pour une grille (lignes × colonnes). Exemple : 60 frames en 2 lignes de 30 réduit la largeur totale de moitié.

#### c) Hauteur de la sprite
-   Hauteur d’une frame × nombre de lignes.
-   Pas de limite aussi stricte qu’en largeur (une image de 200×30 000 px peut poser des problèmes de mémoire). Visez une hauteur sous 8192 px.

#### d) Poids de la sprite
-   **Cible** : < 500 Ko. En dessous de 200 Ko, c’est excellent.
-   Réduire le poids : compresser en PNG (oxipng), diminuer la résolution des frames (100 px au lieu de 200 px), réduire le nombre de couleurs (posterisation), utiliser un fond non transparent pour profiter du JPEG (mais perte de transparence).

#### e) Tableau de correspondances pratiques

| Taille d’affichage CSS par frame | DPR conseillé pour la sprite | Résolution native d’une frame | Nb max de frames (largeur < 10 000 px) | Poids estimé (30 frames, PNG compressé) |
| :--- | :---: | :---: | :---: | :--- |
| 128 px | 1× | 128 px | 78 | ~120 Ko |
| 200 px | 1× | 200 px | 50 | ~280 Ko |
| 300 px | 1× | 300 px | 33 | ~450 Ko |
| 400 px | 1× | 400 px | 25 | ~700 Ko (⚠️) |
| 200 px | 2× | 400 px | 25 | ~500 Ko |
| 100 px | 2× | 200 px | 50 | ~200 Ko |

> *Note : ces poids sont indicatifs et dépendent du contenu de l’image. Une sprite avec beaucoup de détails pèsera davantage.*

### 4.5. Textures et fonds répétitifs

-   **Taille des tuiles** : entre 64 px et 512 px selon la finesse du motif.
-   **Résolution** : une tuile en 2× (ex. 256 px pour 128 px CSS) assure la netteté sur tous les écrans. Gardez le fichier léger (< 20 Ko par texture).
-   **Animation** : une texture animée par sprite sheet est possible si tuile petite (ex. 128 px). Ex. un motif de 16 frames → sprite de 2048 px de large, très correct.

---

## 5. Tableau récapitulatif général

| Usage | Taille d’affichage recommandée (CSS) | Résolution native préconisée | Format(s) idéal(aux) | Poids cible |
| :--- | :--- | :--- | :--- | :--- |
| **Favicon animé** | 16×16 px (onglet) à 48×48 px | 32×32 px (min), 64×64 px | PNG, GIF, WebP | < 10 Ko |
| **Icone de menu / loader** | 24–64 px | 48–128 px (2×) | PNG, SVG (si vectoriel) | < 20 Ko |
| **Logo / objet 3D (fallback)** | 150–400 px | 300–800 px | PNG, APNG, WebP, sprite sheet | < 200 Ko (statique), < 400 Ko (animé) |
| **Illustration de héros** | 800–1200 px | 1600–2400 px (2×) | JPEG, WebP, vidéo | < 500 Ko |
| **Sprite sheet (petite)** | 100–150 px / frame | 100–150 px (1×) | PNG (sprite sheet) | < 150 Ko |
| **Sprite sheet (moyenne)** | 200–300 px / frame | 200–300 px (1×) | PNG | < 400 Ko |
| **Sprite sheet (grande)** | 400 px+ / frame | à éviter ; utiliser une vidéo | – | – |
| **Fond de page (tuile)** | 100–300 px | 200–600 px (2×) | JPEG, WebP | < 30 Ko |
| **Fond animé plein écran** | 1920×1080 px (ou 100vw) | 1920 px (vidéo) | MP4, WebM | < 1 Mo (vidéo courte) |

---

## 6. Contraintes techniques et mémoire GPU

-   **Taille maximale d’image** : 16 384 px × 16 384 px (navigateurs basés sur Chromium). Firefox et Safari supportent des tailles similaires, mais la mémoire allouée devient critique.
-   **Mémoire GPU** : une image de 16 384×16 384 px en RGBA occupe 1 Go de mémoire vidéo. C’est inacceptable. Gardez les images sous les 4096×4096 px pour une compatibilité sereine (16 Mo). Pour les sprite sheets, une largeur 10 000 px × hauteur 200 px ne pèse « que » 8 Mo en RGBA, ce qui est acceptable.
-   **Bande passante** : pensez aux utilisateurs mobiles. Une image de 1 Mo peut coûter cher en données. Utilisez toujours une compression efficace et servez des images adaptatives.

---

## 7. Adaptation responsive et haute densité

### 7.1. Images statiques / animées avec `<img>` ou `<picture>`

Utilisez l’attribut `srcset` pour proposer différentes résolutions :

```html
<img src="image-1x.png"
     srcset="image-1x.png 1x, image-2x.png 2x, image-3x.png 3x"
     alt="Objet animé"
     style="width:200px; height:auto;">
```

Le navigateur choisit la meilleure résolution selon son DPR.

### 7.2. Sprite sheets et media queries

Pour une sprite sheet, on peut charger une version 2× sur les écrans haute densité via une règle CSS :

```css
.sprite {
  background-image: url('sprite-1x.png');
}
@media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
  .sprite {
    background-image: url('sprite-2x.png');
    background-size: auto 100%; /* Ajuste la taille à l'échelle */
  }
}
```

Cependant, le doublement de la résolution multiplie par 4 le poids et la largeur de la sprite. Une alternative pragmatique est de **toujours utiliser la version 1×** et de l’afficher avec `image-rendering: auto` (l’interpolation du navigateur est suffisante pour les animations).

### 7.3. Canvas et adaptation

Si vous utilisez un canvas pour dessiner une sprite sheet, définissez sa dimension en pixels physiques via `canvas.width = canvas.offsetWidth * devicePixelRatio`, puis appliquez une mise à l’échelle du contexte. Cela rendra l’image nette sur écrans Retina, mais augmentera la charge de dessin. Pour une animation simple, restez en 1×.

---

## 8. Accessibilité et performance

-   **Respect de `prefers-reduced-motion`** : désactivez les animations lourdes (remplacement par image statique).
-   **Lazy loading** : ne chargez les images animées que si elles sont visibles dans la fenêtre (`loading="lazy"` sur `<img>`, `IntersectionObserver` pour les canvas).
-   **Poids progressifs** : utilisez des images progressives ou un flou d’attente pour les sprite sheets (basse résolution d’abord, puis la vraie).
-   **Cache** : configurez des en‑têtes HTTP de cache longs pour les images d’animation qui ne changent pas.

---

## 9. Checklist finale

Avant d’intégrer une image ou une animation, vérifiez :

- [ ] La taille d’affichage CSS est définie et correspond à l’usage.
- [ ] La résolution native de l’image est adaptée au DPR cible sans excès.
- [ ] Pour une sprite sheet : largeur totale sous 10 000 px, poids < 500 Ko.
- [ ] Des alternatives sont prévues pour les écrans haute densité (si nécessaire).
- [ ] Le format est bien choisi (PNG pour transparence, WebP pour compression, vidéo pour fonds larges).
- [ ] L’animation respecte les règles de `prefers-reduced-motion`.
- [ ] Un fallback statique est présent en cas d’échec de chargement.

---

## 10. Conclusion

Le dimensionnement des images d’animation est un équilibre subtil entre netteté, fluidité et performance. En appliquant les principes de ce guide – résolution juste nécessaire, compression efficace, formats modernes – vous offrirez une expérience visuelle de qualité à tous vos visiteurs, quel que soit leur appareil.

Vous avez désormais toutes les cartes en main pour dimensionner correctement vos sprite sheets, fallbacks 3D, icônes animées et fonds décoratifs.