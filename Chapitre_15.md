# Chapitre 15 — Niveaux et briques incassables

[« Précédent](Chapitre_14.md) | [Accueil](index.md) | [Suivant »](Chapitre_16.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Sortir de la grille uniforme : définir des **niveaux** dessinés à partir d'un **plan**
(un *pattern*), et introduire des **briques incassables** qui renvoient la balle sans
jamais se briser.

---

## Décrire un niveau avec un plan

Plutôt que de coder chaque niveau « en dur », on le décrit avec une grille de **codes**,
un par emplacement de brique :

```
 0 = rien (vide)
 1 = brique normale
 2 = brique solide (2 coups)
 9 = brique incassable
```

Un niveau, c'est donc une liste de codes (une ligne par rangée) :

```cpp
struct Level {
    int rows;
    std::vector<int> plan;    // BRICK_ROWS * BRICK_COLS codes
};
```

Exemple de plan (9 colonnes, 4 rangées) — un cadre incassable autour de briques
normales :

```
 9 9 9 9 9 9 9 9 9
 9 1 1 1 1 1 1 1 9
 9 1 2 2 2 2 2 1 9
 9 9 9 9 9 9 9 9 9
```

---

## La brique gagne un drapeau « incassable »

On enrichit la structure `Brick` du chapitre 11 :

```cpp
struct Brick {
    Rect     r;
    uint16_t color;
    int      hp;
    bool     alive;
    bool     indestructible;   // <-- nouveau
};
```

---

## Construire les briques à partir du plan

On parcourt le plan et on crée une brique selon le code (on saute le code 0) :

```cpp
std::vector<Brick> build_level(const Level& lvl) {
    std::vector<Brick> out;
    for (int j = 0; j < lvl.rows; j++) {
        for (int i = 0; i < BRICK_COLS; i++) {
            int code = lvl.plan[j * BRICK_COLS + i];    // j*w+i, encore l'adressage !
            if (code == 0) continue;                    // vide : pas de brique

            int x = BRICK_LEFT + i * (BRICK_W + BRICK_GAP);
            int y = BRICK_TOP  + j * (BRICK_H + BRICK_GAP);

            Brick b;
            b.r = { x, y, BRICK_W, BRICK_H };
            b.alive = true;
            b.indestructible = (code == 9);
            b.hp    = (code == 9) ? 9999 : code;        // 1 ou 2 coups
            b.color = (code == 9) ? color_gray
                    : (code == 2) ? color_red
                    :               color_orange;
            out.push_back(b);
        }
    }
    return out;
}
```

---

## Les 3 règles à ne PAS oublier pour les incassables

C'est là que se cachent les bugs. Une brique incassable doit :

**1. Rebondir sans dégât ni casse.** Dans la collision (chapitre 11), on teste le drapeau
avant d'infliger des dégâts :

```cpp
if (b.indestructible) {
    // rebond seul, aucun dégât
    sfx.play(880, 30, 0.5f);        // petit "clink"
} else {
    if (--b.hp <= 0) { b.alive = false; score += 100; }
}
// (le calcul de l'axe de rebond reste identique dans les deux cas)
```

**2. Être ignorée pour la fin de niveau.** Sinon le niveau ne se termine **jamais** (il
reste toujours des incassables). On corrige le test du chapitre 11 :

```cpp
bool level_cleared(const std::vector<Brick>& bricks) {
    for (auto& b : bricks)
        if (b.alive && !b.indestructible)   // on ne compte que les CASSABLES
            return false;
    return true;
}
```

**3. Ne pas être détruite par le laser** (chapitre 14) : le tir s'arrête dessus mais ne
la casse pas.

```cpp
if (b.alive && overlap(s.r, b.r)) {
    if (!b.indestructible && --b.hp <= 0) { b.alive = false; g.score += 100; }
    s.active = false;                       // le tir s'arrête dans tous les cas
    break;
}
```

---

## Sécurité : jamais un niveau 100 % incassable

Un plan entièrement composé de `9` serait **impossible à finir**. Quand tu génères des
niveaux (chapitre 16), garantis **au moins une** brique cassable — par exemple en
forçant une rangée normale si le compte de cassables tombe à zéro.

**À tester :** les briques grises renvoient la balle sans se casser, le laser ne les
entame pas, et le niveau se termine **quand même** dès que les briques normales sont
détruites.

---

## À retenir

- Un niveau se décrit par un **plan** de codes ; `build_level` en fabrique les briques.
- Les **incassables** : rebond **sans dégât**, **ignorées** pour la fin de niveau,
  **insensibles** au laser.
- Toujours garantir **au moins une** brique cassable, sinon niveau infini.

---

[« Précédent](Chapitre_14.md) | [Accueil](index.md) | [Suivant » : Génération procédurale](Chapitre_16.md)
