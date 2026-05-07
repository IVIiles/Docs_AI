# 🎞️ Guide Complet des Sprite Sheets pour le Web

> **Version :** 1.0  
> **Date :** 2026-05-07  
> **Objectif :** Maîtriser l’art des sprite sheets d’animation, de la création à l’intégration, en passant par l’optimisation et l’accessibilité.

---

## 1. Qu’est-ce qu’une sprite sheet d’animation ?

Une **sprite sheet** (ou planche de sprites) est une image unique contenant toutes les images‑clés d’une animation, disposées côte à côte (ou en grille). En affichant successivement chaque portion de l’image, on recrée l’illusion du mouvement.

Les sprite sheets sont utilisées depuis les débuts du jeu vidéo et restent une méthode robuste pour les animations web :
-   Une seule requête HTTP (contrairement aux multiples images individuelles).
-   Contrôle total via CSS ou JavaScript (pause, inverse, vitesse).
-   Compatibilité universelle (CSS 2.1 minimum).

---

## 2. Terminologie et concepts clés

| Terme | Définition |
| :--- | :--- |
| **Frame** | Une image individuelle de l’animation. |
| **Taux de rafraîchissement (FPS)** | Nombre de frames affichées par seconde. 24 FPS minimum pour un mouvement fluide, 30 FPS recommandé, 60 FPS pour la perfection. |
| **Planche horizontale** | Les frames sont alignées sur une seule ligne. Simple à intégrer, mais limitée en largeur (max ~16 384 px). |
| **Grille** | Les frames occupent plusieurs lignes et colonnes. Permet de dépasser la limite de largeur d’image. |
| **Pas (step)** | Le nombre de positions que l’image de fond doit parcourir. Équivaut au nombre de frames dans une planche horizontale. |
| **Durée de frame** | Temps d’affichage d’une frame = 1 / FPS. Ex. 30 FPS → 33,3 ms par frame. |

---

## 3. Préparer son animation et exporter sa sprite sheet

### 3.1. Choix du nombre de frames et de la durée

-   **30 FPS** est le bon compromis qualité/performance pour le web (une rotation lisse sans poids excessif).
-   Pour une rotation complète en 1 seconde → 30 frames. En 2 secondes → 60 frames.
-   Réduisez le nombre de frames si l’animation est petite (icône) ou si le style assume un effet stroboscopique.

### 3.2. Création depuis Blender (ou autre logiciel 3D)

1.  **Configurer la sortie** :  
    `Output Properties > Format > PNG`, `Color > RGBA` (transparence).  
    `Resolution` adaptée à l’usage (200 × 200 px pour un objet de taille moyenne).
2.  **Cadrer l’objet** :  
    Verrouillez la caméra, activez `Film > Transparent` pour le fond alpha.
3.  **Définir le nombre de frames** :  
    Sur une rotation de 360° en 1 seconde à 30 FPS, réglez la timeline sur 30 images (`Output > Frame Range > End Frame = 30`).
4.  **Rendu en séquence PNG** :  
    `Render > Render Animation` produit 30 fichiers individuels (`0001.png` à `0030.png`).

### 3.3. Assemblage en sprite sheet

#### Option 1 : Assemblage horizontal avec ImageMagick
```bash
montage *.png -tile 30x1 -geometry 200x200+0+0 -background none sprite.png
```
-   `-tile 30x1` : 30 frames en ligne, 1 ligne.
-   `-geometry` : taille d’une frame (doit correspondre à votre résolution).
-   `-background none` : transparence préservée.

#### Option 2 : Grille avec ffmpeg (filmstrip)
```bash
ffmpeg -i sequence_%04d.png -vf "tile=10x3" -frames:v 1 sprite_grid.png
```
-   `tile=10x3` : 10 colonnes, 3 lignes (30 frames au total).

#### Option 3 : Outils graphiques
-   **TexturePacker** (gratuit, avec interface) : importez vos PNG, choisissez la disposition, exportez en sprite sheet.
-   **Glue** (ligne de commande) : simple et efficace.
-   **GIMP / Photoshop** : créez un nouveau document de la taille finale, copiez chaque frame en position.
-   **Plugins Blender** : certains permettent l’export direct en sprite sheet (ex. “Render Animation as Sprite Sheet”).

#### Vérification
-   L’image finale doit mesurer `(largeur frame × nb colonnes)` × `(hauteur frame × nb lignes)`.
-   Chaque frame doit avoir exactement la même taille et le même alignement.

### 3.4. Optimisation du fichier

-   Utilisez **PNG‑8** si moins de 256 couleurs, sinon **PNG‑24** avec compression maximale.
-   **TinyPNG / Squoosh / oxipng** réduisent le poids de 30 à 70 % sans perte visible.
-   Limitez la résolution : 200×200 px par frame est souvent suffisant.  
-   Si la transparence n’est pas indispensable, envisagez un **JPEG progressif** (mais sans alpha).

---

## 4. Intégration CSS : la méthode `steps()`

### 4.1. Principe

On affiche la première frame en fond d’un conteneur, puis on décale la position du fond avec une animation CSS utilisant `animation-timing-function: steps(n)`.

```css
.sprite {
  width: 200px;                    /* largeur d’une frame */
  height: 200px;                   /* hauteur d’une frame */
  background-image: url('sprite.png');
  background-size: auto 100%;      /* laisse la hauteur fixe, largeur auto */
  background-repeat: no-repeat;
  animation: play 1s steps(30) infinite;
}

@keyframes play {
  from { background-position: 0 0; }
  to   { background-position: -6000px 0; }  /* 30 frames × 200px */
}
```

### 4.2. Gestion des grilles

Si les frames sont en grille (ex. 10 colonnes, 3 lignes), il faut animer `background-position` sur les deux axes. La propriété `steps()` n’anime bien qu’une seule valeur ; il faut donc ruser en utilisant deux animations séparées, ou privilégier une planche horizontale.

**Solution recommandée :** Restez en ligne horizontale lorsque c’est possible (moins de 80 frames). Pour les grilles, préférez une approche JavaScript (voir section 5).

### 4.3. Contrôle avancé avec CSS

-   **Mettre en pause / rejouer** : utilisez `animation-play-state`.
-   **Inverser l’animation** : créez un second `@keyframes` avec `background-position` inverse (ou utilisez `animation-direction: reverse`).
-   **Définir une vitesse variable** : modifier `animation-duration` via une variable CSS ou en ajoutant/supprimant une classe.

```css
.sprite.paused {
  animation-play-state: paused;
}
.sprite.fast {
  animation-duration: 0.5s;
}
```

> ⚠️ Une fois l’animation déclenchée, modifier les propriétés peut déclencher un recalcul de style. Pour des changements fréquents, le JS est plus approprié.

---

## 5. Contrôle par JavaScript : l’approche « image de fond » ou « toile »

### 5.1. Mise à jour du `background-position`

On peut directement lire et écrire la position du fond via JS, ce qui permet de synchroniser l’animation avec le scroll, la souris, etc.

```javascript
const el = document.querySelector('.sprite');
const frameWidth = 200;
const totalFrames = 30;
let currentFrame = 0;

function updateFrame(frame) {
  const x = -frame * frameWidth;
  el.style.backgroundPosition = `${x}px 0`;
}

// Avancer d'une frame toutes les 33ms (30 FPS)
setInterval(() => {
  currentFrame = (currentFrame + 1) % totalFrames;
  updateFrame(currentFrame);
}, 33);
```

Avantages : contrôle total, synchronisation avec l’`IntersectionObserver` ou le défilement.

Inconvénients : vous perdez l’optimisation GPU de l’animation CSS `steps()` (mais sur un objet de taille modeste, l’impact est négligeable).

### 5.2. Utilisation d’un `canvas` 2D

Pour aller plus loin, on peut charger la sprite sheet dans un canvas et y dessiner la portion correspondante.

```javascript
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const img = new Image();
img.src = 'sprite.png';

img.onload = () => {
  function drawFrame(frame) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.drawImage(img, frame * frameWidth, 0, frameWidth, frameHeight, 0, 0, canvas.width, canvas.height);
  }
  // boucle d'animation...
};
```

Cette approche permet aussi de teinter, redimensionner ou appliquer des effets sur les frames.

---

## 6. Intégration avec React

### 6.1. Composant simple avec CSS Sprite

```jsx
import sprite from './assets/sprite.png';

function AnimatedSprite({ width = 200, frames = 30, duration = 1 }) {
  return (
    <div
      className="sprite"
      style={{
        width,
        height: width,
        backgroundImage: `url(${sprite})`,
        backgroundSize: `auto 100%`,
        animation: `play ${duration}s steps(${frames}) infinite`,
      }}
    />
  );
}
```

Ajoutez le `@keyframes` dans votre CSS global ou via `styled-components`.

### 6.2. Contrôle réactif avec hooks

```jsx
import { useRef, useEffect, useState } from 'react';

function ControllableSprite({ frames, frameWidth }) {
  const containerRef = useRef(null);
  const [currentFrame, setCurrentFrame] = useState(0);

  // Change la frame selon un état (ex. progrès de lecture)
  useEffect(() => {
    containerRef.current.style.backgroundPosition = `-${currentFrame * frameWidth}px 0`;
  }, [currentFrame]);

  return <div ref={containerRef} className="sprite-base" />;
}
```

### 6.3. Optimisation : préchargement et lazy loading

-   Placez la sprite sheet dans `public/` ou importez‑la pour que Vite la gère (hash + cache).
-   Utilisez `<link rel="preload">` pour les animations critiques.
-   Si l’animation n’est pas dans la fenêtre au chargement, attendez qu’elle devienne visible (`IntersectionObserver`) avant de démarrer l’animation.

---

## 7. Accessibilité et bonnes pratiques

### 7.1. Réduction des mouvements

```css
@media (prefers-reduced-motion: reduce) {
  .sprite {
    animation: none !important;
    background-position: 0 0; /* affiche la première frame */
  }
}
```

### 7.2. Alternative textuelle

Si la sprite sheet est porteuse de sens, ajoutez un texte accessible :

```html
<div class="sprite" role="img" aria-label="Logo 3D tournant"></div>
```

Pour les animations purement décoratives, utilisez `aria-hidden="true"`.

### 7.3. Performance d’affichage

-   Préférez `transform: translateZ(0)` (ou `will-change: background-position`) si vous changez souvent la position via JS, pour pousser l’élément sur un calque GPU.
-   Limitez la taille de l’image : un sprite de 12 000 × 200 px reste raisonnable, mais surveillez le poids total.
-   Pas de `background-size` supérieur à la taille réelle (évitez de demander au navigateur de redimensionner l’image à chaque rendu).

### 7.4. Choix du format d’image final

| Format | Transparence | Couleurs | Poids optimisé | Utilisation typique |
| :--- | :---: | :---: | :---: | :--- |
| **PNG** (sprite sheet) | Oui | 16M | ++ (après oxipng) | Standard |
| **WebP** (sprite statique) | Oui | 16M | +++ | Rare (peu d’outils), mais possible |
| **APNG** (alternative) | Oui | 16M | ++ | Pas une sprite sheet, mais une image animée alternative |
| **SVG** (sprites vectoriels) | Oui | ∞ | + | Uniquement si images vectorielles, pas de photos |

La sprite sheet reste le meilleur compromis pour un contrôle fin avec une compatibilité maximale.

---

## 8. Alternatives modernes et quand les choisir

| Solution | Quand la préférer |
| :--- | :--- |
| **Sprite sheet (CSS)** | Contrôle total nécessaire, compatibilité ultra-large, une seule source d’image. |
| **GIF animé** | Simplicité extrême, pas de contrôle, palette 256 couleurs. |
| **APNG / WebP animé** | Qualité photoréaliste avec transparence, intégration `<img>` la plus simple. |
| **Lottie** | Animations vectorielles complexes exportées depuis After Effects, interactives. |
| **Vidéo (mp4)** | Rendu 3D lourd, pas de transparence nécessaire. |
| **JavaScript + canvas** | Manipulation d’image avancée, effets en temps réel, animations générées dynamiquement. |

---

## 9. Checklist d’une implémentation professionnelle

Avant de livrer votre animation en sprite sheet, vérifiez les points suivants :

- [ ] La planche est bien horizontale (ou la grille est documentée).
- [ ] Chaque frame occupe strictement la même surface (pas de décalage).
- [ ] Le nombre de frames correspond à la durée et au FPS souhaités.
- [ ] Les dimensions de l’image ne dépassent pas 16 384 px de largeur (pour une planche horizontale).
- [ ] Le poids de l’image est optimisé (< 200 Ko idéalement, jamais plus de 500 Ko pour une animation critique).
- [ ] L’animation est désactivée pour les utilisateurs ayant activé `prefers-reduced-motion`.
- [ ] Un fallback statique est prévu si l’image ne charge pas.
- [ ] L’animation se met en pause lorsque l’élément n’est pas visible (`IntersectionObserver`).
- [ ] Le composant React gère proprement le démontage (pas d’animation qui continue en arrière‑plan).
- [ ] Les métadonnées de l’image (alt, role) sont correctement définies.

---

## 10. Résumé opérationnel

-   **Quand utiliser une sprite sheet ?** Dès que vous avez besoin de contrôler précisément une animation photoréaliste (lecture, pause, vitesse) avec une compatibilité maximale, sans ajouter de lourdes bibliothèques.
-   **Comment la créer ?** Rendu image par image depuis Blender → assemblage horizontal avec ImageMagick → compression PNG.
-   **Comment l’intégrer ?** CSS `steps()` pour les boucles simples, JavaScript pour un contrôle fin.
-   **Quelle est la limite ?** Longueur de l’animation : pas plus de ~80 frames en ligne, sinon utiliser une vidéo ou un APNG.

Une sprite sheet bien conçue est une arme redoutable : elle offre des performances prévisibles partout, du vieux PC aux smartphones d’entrée de gamme. Elle mérite sa place dans votre boîte à outils d’animations web.