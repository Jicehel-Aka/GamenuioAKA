# Chapitre 10 — Collision et rebond Arkanoid

[« Précédent](Chapitre_09.md) | [Accueil](index.md) | [Suivant »](Chapitre_11.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Deux problèmes bien distincts, qu'on va traiter **l'un après l'autre** :

1. **Détecter** que la balle touche la raquette (une collision).
2. **Réagir** : calculer le rebond. On commencera par le plus simple, puis on
   l'enrichira étape par étape jusqu'au rebond « Arkanoid » où l'angle dépend de l'endroit
   touché.

---

## Partie 1 — Détecter une collision (générique)

### L'idée : deux rectangles se chevauchent-ils ?

La balle et la raquette sont des **rectangles** (on a créé la primitive `Rect` au
chapitre 8). Deux rectangles se touchent si, **et seulement si**, ils se chevauchent à la
fois **horizontalement** ET **verticalement**.

```
   se chevauchent (contact)         ne se chevauchent pas
   ┌─────┐                          ┌─────┐
   │  A  │                          │  A  │
   │   ┌─┼───┐                      └─────┘
   └───┼─┘ B │                          ┌─────┐
       └─────┘                          │  B  │
                                        └─────┘
```

Regardons **un seul axe** (l'horizontal) : A et B se chevauchent en x si le **bord
gauche** de A est plus à gauche que le **bord droit** de B, **et** inversement :

```
A.x  ........ A.x+A.w
        B.x ........ B.x+B.w
        └── chevauchement ──┘     ⇔   A.x < B.x+B.w   ET   A.x+A.w > B.x
```

On applique la même chose en vertical. Les **quatre** conditions ensemble donnent la
collision. C'est l'algorithme « AABB » (*Axis-Aligned Bounding Box* : boîtes alignées sur
les axes).

### La fonction générique

On l'écrit **une fois**, et elle marchera pour n'importe quelle paire : balle/raquette,
balle/brique, etc. C'est le bénéfice d'avoir une primitive `Rect` commune.

```cpp
bool overlap(const Rect& a, const Rect& b) {
    return a.x < b.x + b.w &&      // bord gauche de A avant bord droit de B
           a.x + a.w > b.x &&      // bord droit de A après bord gauche de B
           a.y < b.y + b.h &&      // bord haut de A au-dessus du bas de B
           a.y + a.h > b.y;        // bord bas de A en-dessous du haut de B
}
```

Comme la balle est en `float`, on lui fabrique un `Rect` au vol :

```cpp
Rect ball_rect(const Ball& b) {
    return { (int)b.x, (int)b.y, b.size, b.size };
}
```

**Test simple :** `if (overlap(ball_rect(ball), paddle.r)) { ... }` → vrai quand la balle
touche la raquette. On tient la détection. Reste à décider **quoi faire**.

---

## Partie 2 — Réagir : le rebond, du plus simple au plus fin

### Niveau 1 — Le rebond plat

Le minimum : quand la balle touche la raquette **en descendant**, on la renvoie vers le
haut. On inverse `vy` (et on la force vers le haut pour être sûr) :

```cpp
if (ball.vy > 0 && overlap(ball_rect(ball), paddle.r)) {
    ball.vy = -ball.vy;                       // renvoyée vers le haut
    ball.y  = paddle.r.y - ball.size - 1;     // on la sort de la raquette
}
```

Le `ball.vy > 0` (balle qui descend) évite de re-rebondir si elle remonte déjà. Le
repositionnement évite les **collisions multiples** (sinon elle resterait « dans » la
raquette plusieurs images de suite).

Ça marche… mais c'est **plat** : la balle repart toujours au même angle. Le joueur ne
peut pas viser. Améliorons.

### Niveau 2 — Des zones sur la raquette

Idée : découper la raquette en **zones**. Toucher à gauche renvoie vers la gauche,
toucher au centre renvoie tout droit, toucher à droite renvoie vers la droite.

```
  raquette découpée en 5 zones :
   ┌─────┬─────┬─────┬─────┬─────┐
   │ -2  │ -1  │  0  │ +1  │ +2  │
   └─────┴─────┴─────┴─────┴─────┘
   gauche         centre         droite
```

On calcule dans **quelle** zone la balle a tapé, et on en déduit `vx` :

```cpp
int zone_impact(const Ball& b, const Paddle& p, int nb_zones) {
    int centre_balle = (int)b.x + b.size / 2;
    int rel = centre_balle - p.r.x;                 // 0 .. largeur raquette
    int zone = rel * nb_zones / p.r.w;              // 0 .. nb_zones-1
    return zone - nb_zones / 2;                     // recentre : ... -1, 0, +1 ...
}

// à la collision :
int z = zone_impact(ball, paddle, 5);   // renvoie -2..+2
ball.vx = z * 1.3f;                      // plus la zone est extrême, plus vx est grand
ball.vy = -fabs(ball.vy);                // toujours vers le haut
```

C'est déjà **beaucoup** plus jouable : on peut orienter la balle. Mais la vitesse totale
change selon la zone (une balle très inclinée va plus « vite » en diagonale). Affinons
encore.

### Niveau 3 — Un pourcentage continu

Au lieu de 5 zones, utilisons une valeur **continue** `t` entre **-1** (bord gauche) et
**+1** (bord droit), avec **0** au centre exact :

```cpp
float impact_ratio(const Ball& b, const Paddle& p) {
    float centre_balle   = b.x + b.size / 2.0f;
    float centre_raquette = p.r.x + p.r.w / 2.0f;
    float t = (centre_balle - centre_raquette) / (p.r.w / 2.0f);  // -1 .. +1
    if (t < -1) t = -1;
    if (t >  1) t =  1;
    return t;
}
```

```
   bord gauche      centre      bord droit
       t = -1        t = 0         t = +1
        │─────────────│─────────────│
```

On peut alors moduler la vitesse en douceur :

```cpp
float t = impact_ratio(ball, paddle);
ball.vx = t * 3.0f;
ball.vy = -fabs(ball.vy);
```

C'est fluide et intuitif. Il reste un dernier raffinement, facultatif, si tu veux la
sensation exacte d'Arkanoid : **garder une vitesse totale constante** en ne faisant que
**tourner** la direction.

### Niveau 4 (bonus) — Un vrai angle, à vitesse constante (le sinus, expliqué)

Ici, et seulement ici, on sort la trigonométrie — mais on va comprendre **pourquoi**.

On veut : centre touché → balle **tout droit vers le haut** ; bord touché → balle
renvoyée **en biais**, jusqu'à un angle maximal (disons 60°). Notre `t` (-1..+1) donne
justement « à quel point on est vers le bord ». On le transforme en **angle** :

```
angle = t × 60°     (t=-1 → -60°,  t=0 → 0°,  t=+1 → +60°)
```

Maintenant, comment transformer un **angle** en une **vitesse (vx, vy)** ? C'est le rôle
du **sinus** et du **cosinus**. Sur un cercle de rayon = la vitesse, un angle mesuré
**depuis la verticale** (le « tout droit vers le haut ») se décompose ainsi :

```
            ↑ (tout droit, angle 0)
            │
        \   │   /
         \  │  /   ← direction de sortie, inclinée de "angle"
          \ │ /
   ────────●────────
       vx = vitesse × sin(angle)      (composante horizontale)
       vy = vitesse × cos(angle)      (composante verticale, vers le haut)
```

La propriété clé : pour **n'importe quel** angle, `sin(angle)² + cos(angle)² = 1`. Donc
la **longueur** du vecteur (vx, vy) — c'est-à-dire la **vitesse totale** — reste égale à
`vitesse`, quel que soit l'angle. On ne fait que **pivoter** la direction, sans
accélérer ni ralentir. C'est exactement l'effet Arkanoid.

```cpp
#include <cmath>

void ball_vs_paddle(Ball& b, const Paddle& p) {
    if (b.vy <= 0 || !overlap(ball_rect(b), p.r)) return;

    float t = impact_ratio(b, p);            // -1 .. +1  (niveau 3)
    float angle_max = 1.05f;                 // ~60° en radians
    float angle = t * angle_max;

    float vitesse = std::sqrt(b.vx*b.vx + b.vy*b.vy);   // la vitesse actuelle
    b.vx =  vitesse * std::sin(angle);       // composante horizontale
    b.vy = -vitesse * std::cos(angle);       // composante verticale (vers le haut)

    b.y = p.r.y - b.size - 1;                // on la sort de la raquette
}
```

> Deux mots sur les **radians** : `sin`/`cos` du C++ attendent l'angle en radians, pas en
> degrés. 180° = π ≈ 3,14159 rad ; donc 60° ≈ 1,05 rad. C'est pour ça que `angle_max`
> vaut `1.05f`.

---

## Choisis ton niveau

Tu n'es **pas** obligé d'aller jusqu'au sinus. Chaque niveau est un jeu qui marche :

| Niveau | Sensation | Complexité |
|---|---|---|
| 1 · rebond plat | robotique | triviale |
| 2 · zones | on peut viser | facile |
| 3 · pourcentage | fluide | facile |
| 4 · sinus | vrai Arkanoid, vitesse constante | trigo de base |

Le but pédagogique : **assimiler la brique de base, puis la rendre générique et la
raffiner par couches**. C'est comme ça qu'on progresse sans copier de « code magique ».

---

## À retenir

- **Détecter** et **réagir** sont deux problèmes séparés : traite-les l'un après l'autre.
- La collision **AABB** (`overlap`) est **générique** : une fonction pour toutes les
  paires de `Rect`.
- Le rebond se construit **progressivement** ; le sinus n'est qu'une **dernière couche**
  optionnelle pour garder une vitesse constante — et il se comprend avec un simple cercle.

---

[« Précédent](Chapitre_09.md) | [Accueil](index.md) | [Suivant » : Les briques](Chapitre_11.md)
