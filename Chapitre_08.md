# Chapitre 08 — La raquette

[« Précédent](Chapitre_07.md) | [Accueil](index.md) | [Suivant »](Chapitre_09.md)

![Niveau](https://img.shields.io/badge/Niveau-D%C3%A9butant-green)

---

## Objectif

Créer un premier objet du jeu, la **raquette**, qu'on déplace au joystick et aux flèches,
et qui reste dans l'écran. Au passage, on introduit deux outils C++ essentiels : la
**structure** et une **primitive réutilisable**.

---

## D'abord : c'est quoi une structure (`struct`) ?

Un objet du jeu, c'est plusieurs informations qui vont ensemble. La raquette, par
exemple, a une **position** et une **taille**. Plutôt que de trimballer quatre variables
séparées (`px, py, pw, ph`), on les **regroupe** dans une **structure** :

```cpp
struct Rect {     // "Rect" = un rectangle : position + taille
    int x;        // coin haut-gauche, colonne
    int y;        // coin haut-gauche, ligne
    int w;        // largeur
    int h;        // hauteur
};
```

Une `struct`, c'est donc un **nouveau type** qui contient plusieurs champs. On l'utilise
comme n'importe quelle variable :

```cpp
Rect r;                     // on crée un rectangle
r.x = 10; r.y = 20;         // on accède à un champ avec un point
r.w = 48; r.h = 6;
```

> 💡 Pourquoi c'est malin ici : une **raquette**, une **balle**, une **brique** ont
> toutes une position et une taille. Ce sont, géométriquement, des **rectangles**. En
> définissant `Rect` une fois, on va pouvoir le réutiliser pour tout le monde — et,
> surtout, écrire **une seule** fonction de collision qui marche pour n'importe quelle
> paire de rectangles (chapitre 10). C'est le principe du développement « en couches » :
> une petite brique de base, réutilisée partout.

*(Note : on reste en structures simples, très lisibles. Le C++ permet des hiérarchies de
classes avec héritage, mais pour apprendre, des `struct` claires font parfaitement
l'affaire — et restent proches du C.)*

---

## La raquette = un `Rect` + une vitesse

La raquette est un rectangle qui se déplace horizontalement. On lui ajoute donc sa
**vitesse** (de combien de pixels elle avance par image) :

```cpp
constexpr int SCREEN_W = 320, SCREEN_H = 240;
constexpr int PADDLE_W = 48, PADDLE_H = 6;
constexpr int PADDLE_Y = SCREEN_H - 20;    // près du bas
constexpr int PADDLE_SPEED = 5;            // pixels par image

struct Paddle {
    Rect r;                                // position + taille (réutilise Rect)
};

// on crée la raquette centrée, posée en bas
Paddle paddle = { { (SCREEN_W - PADDLE_W) / 2, PADDLE_Y, PADDLE_W, PADDLE_H } };
```

`constexpr` veut dire « constante connue à la compilation » : une valeur fixe à laquelle
on donne un nom lisible. Changer la difficulté = changer un de ces nombres.

---

## Déplacer la raquette

On lit les entrées (chapitre 7) et on ajuste `r.x` :

```cpp
void paddle_update(Paddle& p, const Keys& k) {
    // clavier : maintien => déplacement continu
    if (k.left)  p.r.x -= PADDLE_SPEED;
    if (k.right) p.r.x += PADDLE_SPEED;

    // joystick avec zone morte
    if (k.jx >  200) p.r.x += PADDLE_SPEED;
    if (k.jx < -200) p.r.x -= PADDLE_SPEED;

    // rester dans l'écran (voir ci-dessous)
    if (p.r.x < 0) p.r.x = 0;
    if (p.r.x > SCREEN_W - PADDLE_W) p.r.x = SCREEN_W - PADDLE_W;
}
```

### Le *clamping* (borner une valeur), expliqué

Sans les deux dernières lignes, la raquette sortirait de l'écran. « Borner » (*clamp*),
c'est forcer une valeur à rester entre un minimum et un maximum :

```
   x trop à gauche        x correct           x trop à droite
        x < 0        0 ≤ x ≤ (320 - largeur)     x > 320-largeur
         │                   │                        │
     on remet 0        on ne touche pas        on remet le max
```

- Si `x` devient négatif → on le remet à `0` (bord gauche).
- Si `x` dépasse `SCREEN_W - PADDLE_W` → on le remet à cette valeur (le coin **droit** de
  la raquette touche alors le bord droit).

C'est exactement ce que fait `std::clamp(x, min, max)` de la bibliothèque standard, mais
en l'écrivant à la main on voit ce qui se passe.

---

## Dessiner la raquette

On sait faire depuis le chapitre 4 : un rectangle plein.

```cpp
void paddle_draw(const Paddle& p) {
    gfx.setColor(color_white);
    gfx.fillRect(p.r.x, p.r.y, p.r.w, p.r.h);
}
```

---

## L'assemblage dans la boucle

```cpp
while (true) {
    uint32_t debut = gb.get_millis();

    Keys k;
    read_input(k);              // 1. LIRE (chapitre 7)
    paddle_update(paddle, k);   // 2. METTRE À JOUR

    gfx.clear(color_black);     // 3. DESSINER
    paddle_draw(paddle);
    gfx.update();

    uint32_t travail = gb.get_millis() - debut;   // régulation (chapitre 6)
    if (travail < FRAME_MS) gb.delay_ms(FRAME_MS - travail);
}
```

**À tester :** la raquette suit le joystick et les flèches, et **s'arrête net** aux deux
bords sans jamais sortir.

---

## À retenir

- Une **`struct`** regroupe des variables liées sous un seul nom.
- On définit une **primitive `Rect`** (x, y, w, h) qu'on réutilisera pour la balle et les
  briques — et pour la collision générique du chapitre 10.
- **Borner** (`clamp`) une valeur = la forcer entre un min et un max ; ici pour garder la
  raquette à l'écran.

---

[« Précédent](Chapitre_07.md) | [Accueil](index.md) | [Suivant » : La balle](Chapitre_09.md)
