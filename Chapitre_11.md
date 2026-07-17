# Chapitre 11 — Les briques

[« Précédent](Chapitre_10.md) | [Accueil](index.md) | [Suivant »](Chapitre_12.md)

![Niveau](https://img.shields.io/badge/Niveau-Intermédiaire-yellow)

---

## Objectif

Créer les **briques** du casse‑briques : un tableau de rectangles, chacun avec une couleur
et un état « vivant / détruit ». On introduit la notion de **grille**, et on réutilise
notre collision générique du chapitre 10.

---

## 📸 Illustration
![briques](img/chap11_briques.png)

---

# 📚 Table des matières
- [Structure d’une brique](#structure-dune-brique)
- [Générer une grille](#générer-une-grille)
- [Dessiner les briques](#dessiner-les-briques)
- [Collision balle/brique](#collision-ballebrique)
- [Supprimer une brique](#supprimer-une-brique)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🧱 Structure d’une brique

Une brique est un rectangle (`Rect`) + une couleur + un état :

```cpp
struct Brick {
    Rect r;
    uint16_t color;
    bool alive;
};
```

🧩 Générer une grille
On crée une grille de 6 rangées × 9 colonnes :

```cpp
constexpr int BRICK_W = 30;
constexpr int BRICK_H = 12;
constexpr int BRICK_COLS = 9;
constexpr int BRICK_ROWS = 6;
constexpr int BRICK_GAP  = 2;
constexpr int BRICK_TOP  = 40;

std::vector<Brick> bricks;

void bricks_init() {
    bricks.clear();
    for (int j = 0; j < BRICK_ROWS; j++) {
        for (int i = 0; i < BRICK_COLS; i++) {
            int x = i * (BRICK_W + BRICK_GAP) + 20;
            int y = BRICK_TOP + j * (BRICK_H + BRICK_GAP);
            bricks.push_back({ {x, y, BRICK_W, BRICK_H}, color_orange, true });
        }
    }
}
```

# 🎨 Dessiner les briques
```cpp
void bricks_draw() {
    for (auto& b : bricks) {
        if (!b.alive) continue;
        gfx.setColor(b.color);
        gfx.fillRect(b.r.x, b.r.y, b.r.w, b.r.h);
    }
}
```

# 💥 Collision balle/brique
On réutilise overlap() du chapitre 10 :

```cpp
void ball_vs_bricks(Ball& ball) {
    Rect br = ball_rect(ball);

    for (auto& b : bricks) {
        if (!b.alive) continue;
        if (!overlap(br, b.r)) continue;

        b.alive = false;          // brique détruite
        ball.vy = -ball.vy;       // rebond simple
        return;                   // une seule brique par image
    }
}
```

# 🧹 Supprimer une brique
On ne retire pas la brique du vecteur : on met alive = false.
C’est plus simple, plus rapide, et évite les suppressions en plein jeu.

# 🧠 À retenir
Une brique = un Rect + une couleur + un état.

Une grille = deux boucles imbriquées.

La collision générique overlap() marche pour tout.

On ne supprime pas les briques : on les marque « mortes ».

# 🚀 Pour aller plus loin
Ajouter plusieurs couleurs selon la rangée.

Ajouter des briques à plusieurs points de vie.

Ajouter des briques qui lâchent des bonus (chapitre 14).

# 🧭 Navigation
[Accueil](index.md) | [Suivant »](Chapitre_12.md)
