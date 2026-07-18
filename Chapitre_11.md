# Chapitre 11 — Les briques

[« Précédent](Chapitre_10.md) | [Accueil](index.md) | [Suivant »](Chapitre_12.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Remplir le haut de l'écran de **briques** à casser. On découvre au passage un outil C++
très pratique — le **`std::vector`** (un tableau qui grandit tout seul) — et on
**réutilise** la collision générique du chapitre 10.

---

## Une brique = un `Rect` + des infos

On repart de notre primitive `Rect`. Une brique, c'est un rectangle, plus quelques
informations : sa couleur, ses **points de vie** (`hp`, pour les briques à casser en
plusieurs coups) et un drapeau « vivante / cassée ».

```cpp
struct Brick {
    Rect     r;        // position + taille (chapitre 8)
    uint16_t color;    // sa couleur
    int      hp;       // points de vie : nombre de coups avant destruction
    bool     alive;    // encore là ?
};
```

---

## Stocker les briques : `std::vector`

On ne sait pas toujours à l'avance combien de briques il y aura (ça dépend du niveau).
Un **`std::vector`** est un tableau **dont la taille peut varier** : on y ajoute des
éléments avec `push_back(...)`, on connaît sa taille avec `.size()`, et on le parcourt
comme un tableau.

```cpp
#include <vector>
std::vector<Brick> bricks;     // une liste de briques, vide au départ
```

> 💡 Différence avec le tableau classique (`Brick bricks[54];`) : le `vector` gère la
> mémoire pour toi et retient sa taille. Idéal quand le nombre d'éléments change.

---

## Générer la grille

Les briques forment une **grille** : des rangées et des colonnes. On la construit avec…
deux boucles imbriquées (comme le rectangle du chapitre 4, mais cette fois on place des
briques au lieu de pixels).

```cpp
constexpr int BRICK_W = 30, BRICK_H = 12;   // taille d'une brique
constexpr int BRICK_COLS = 9, BRICK_ROWS = 6;
constexpr int BRICK_GAP  = 2;               // espace entre briques
constexpr int BRICK_TOP  = 40;              // hauteur de la 1re rangée
constexpr int BRICK_LEFT = 20;              // marge à gauche

void bricks_init() {
    bricks.clear();                          // on repart d'une liste vide
    for (int j = 0; j < BRICK_ROWS; j++) {           // chaque rangée
        for (int i = 0; i < BRICK_COLS; i++) {       // chaque colonne
            int x = BRICK_LEFT + i * (BRICK_W + BRICK_GAP);
            int y = BRICK_TOP  + j * (BRICK_H + BRICK_GAP);
            bricks.push_back({ {x, y, BRICK_W, BRICK_H}, color_orange, 1, true });
        }
    }
}
```

```
   colonnes i →   0    1    2    3    4    5    6    7    8
   rangée j=0    [ ][ ][ ][ ][ ][ ][ ][ ][ ]      x = LEFT + i*(W+GAP)
   rangée j=1    [ ][ ][ ][ ][ ][ ][ ][ ][ ]      y = TOP  + j*(H+GAP)
   rangée j=2    [ ][ ][ ][ ][ ][ ][ ][ ][ ]
```

---

## Dessiner les briques

On parcourt le vector et on dessine celles qui sont encore vivantes. La boucle
`for (auto& b : bricks)` veut dire « pour chaque brique `b` de la liste » (le `&` évite
de copier chaque brique) :

```cpp
void bricks_draw() {
    for (auto& b : bricks) {
        if (!b.alive) continue;              // on saute les cassées
        gfx.setColor(b.color);
        gfx.fillRect(b.r.x, b.r.y, b.r.w, b.r.h);
    }
}
```

---

## Collision balle / brique : trouver le bon axe de rebond

On sait **détecter** le contact avec `overlap()` (chapitre 10). Mais quel rebond ? Si la
balle arrive par le **côté**, il faut inverser `vx` ; si elle arrive par le **haut ou le
bas**, il faut inverser `vy`. Un simple `vy = -vy` (comme dans une première version
naïve) est faux dès qu'on touche une brique par le flanc.

L'astuce : comparer de **combien** la balle pénètre horizontalement vs verticalement. La
plus **petite** pénétration indique par où elle est entrée, donc l'axe à inverser.

```
   pénétration horizontale (ox) petite  →  entrée par le côté  →  inverser vx
   pénétration verticale   (oy) petite  →  entrée par le haut  →  inverser vy

        ┌───────────┐
        │  brique   │     • = centre de la balle
     •──┤           │     dx, dy = distance entre les 2 centres
        └───────────┘
```

```cpp
void ball_vs_bricks(Ball& ball, int& score) {
    Rect br = ball_rect(ball);
    for (auto& b : bricks) {
        if (!b.alive) continue;
        if (!overlap(br, b.r)) continue;         // pas de contact : suivante

        // dégâts : --b.hp enlève 1 point de vie ET donne la nouvelle valeur à tester
        if (--b.hp <= 0) { b.alive = false; score += 100; }   // 0 pv => cassée

        // axe de rebond = plus petite pénétration
        float dx = (ball.x + ball.size/2.0f) - (b.r.x + b.r.w/2.0f);
        float dy = (ball.y + ball.size/2.0f) - (b.r.y + b.r.h/2.0f);
        float ox = (b.r.w + ball.size)/2.0f - fabs(dx);   // chevauchement en x
        float oy = (b.r.h + ball.size)/2.0f - fabs(dy);   // chevauchement en y
        if (ox < oy) ball.vx = -ball.vx;                  // entrée par le côté
        else         ball.vy = -ball.vy;                  // entrée par le haut/bas
        break;                                            // une brique par image suffit
    }
}
```

Le `break` s'arrête à la première brique touchée : traiter plusieurs collisions dans la
même image donnerait des rebonds incohérents.

---

## Marquer « cassée », pas supprimer

On met `alive = false` au lieu de retirer la brique du vector. Pourquoi ? Supprimer un
élément **pendant** qu'on parcourt la liste est une source classique de bugs (on peut
sauter un élément ou lire une case qui n'existe plus). Marquer « cassée » est plus simple,
plus rapide, et la brique cesse d'être dessinée et testée.

---

## Le niveau est-il fini ?

Le niveau est terminé quand il ne reste **aucune brique vivante** :

```cpp
bool level_cleared() {
    for (auto& b : bricks)
        if (b.alive) return false;   // il en reste une → pas fini
    return true;                     // aucune vivante → gagné
}
```

*(On adaptera ce test au chapitre 15 pour ignorer les briques **incassables**.)*

**À tester :** la balle casse les briques, rebondit correctement selon l'angle d'arrivée
(côté vs dessus), le score monte, et le niveau se termine quand tout est cassé.

---

## À retenir

- Un **`std::vector`** est un tableau extensible : `push_back`, `.size()`, parcours
  `for (auto& x : v)`.
- La collision réutilise **`overlap()`** ; le **bon axe de rebond** se trouve en comparant
  les pénétrations horizontale et verticale.
- On **marque** les briques cassées (`alive=false`), on ne les supprime pas en plein jeu.

---

[« Précédent](Chapitre_10.md) | [Accueil](index.md) | [Suivant » : Machine à états](Chapitre_12.md)
