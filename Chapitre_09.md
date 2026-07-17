# Chapitre 09 — La balle

[« Précédent](Chapitre_08.md) | [Accueil](index.md) | [Suivant »](Chapitre_10.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Faire vivre une **balle** : lui donner une position **et une vitesse**, la faire avancer,
et la faire **rebondir** sur les murs. C'est notre première « physique ».

---

## Position + vitesse = mouvement

La raquette avait une position. La balle a en plus une **vitesse**, c'est-à-dire de
combien elle se déplace **par image**, en x et en y :

```cpp
struct Ball {
    float x, y;       // position (en flottant : voir plus bas)
    float vx, vy;     // vitesse : déplacement par image, en x et en y
    int   size;       // côté de la balle (un petit carré, pour commencer)
    bool  active;     // la balle est-elle en jeu ?
};
```

À chaque image, **avancer = ajouter la vitesse à la position** :

```cpp
b.x += b.vx;      // b.x = b.x + b.vx
b.y += b.vy;
```

```
   position à l'image N        position à l'image N+1
        (x, y)      --- + (vx, vy) --->   (x+vx, y+vy)
```

### Pourquoi des `float` (nombres à virgule) ?

Une vitesse comme « 2,6 pixels par image » n'est pas un entier. Si on stockait la
position en entiers, on perdrait ces fractions et les rebonds seraient imprécis (angles
qui « collent » aux axes). On calcule donc en **`float`** (nombres à virgule), et on
n'arrondit **qu'au dernier moment**, pour dessiner :

```cpp
gfx.setColor(color_yellow);
gfx.fillRect((int)b.x, (int)b.y, b.size, b.size);   // (int) = on arrondit pour l'écran
```

`(int)b.x` transforme le flottant en entier (il coupe la partie décimale). L'écran ne
connaît que des pixels entiers, mais la **physique**, elle, garde toute sa précision.

---

## Rebondir sur un mur = inverser une vitesse

Un mur vertical (gauche/droite) renvoie la balle **horizontalement** : on inverse `vx`.
Un mur horizontal (le plafond) renvoie **verticalement** : on inverse `vy`. « Inverser »
= changer le signe.

```
   avant :  balle → mur │            après :  │ mur ← balle
            vx = +2,6                          vx = -2,6
```

```cpp
constexpr int SCREEN_W = 320, SCREEN_H = 240;

void ball_update(Ball& b) {
    if (!b.active) return;

    b.x += b.vx;
    b.y += b.vy;

    // mur gauche
    if (b.x < 0)                 { b.x = 0;                 b.vx = -b.vx; }
    // mur droit
    if (b.x > SCREEN_W - b.size) { b.x = SCREEN_W - b.size; b.vx = -b.vx; }
    // plafond
    if (b.y < 0)                 { b.y = 0;                 b.vy = -b.vy; }
    // le bas = balle perdue : on s'en occupe au chapitre "états"
}
```

Remarque le double geste à chaque mur : on **replace** d'abord la balle exactement sur le
bord (`b.x = 0`, etc.), **puis** on inverse la vitesse. Sans le replacement, la balle
pourrait rester « coincée » un peu au-delà du mur et re-déclencher le rebond à l'image
suivante (elle vibrerait sur place).

---

## Lancer la balle

Pour tester tout de suite, on lance la balle en diagonale au démarrage :

```cpp
Ball ball = { 160.0f, 120.0f, 2.6f, -2.6f, 6, true };
//             x       y       vx     vy   size active
```

`vx = 2.6` (vers la droite) et `vy = -2.6` (vers le haut) : la balle part en diagonale
haut-droite.

---

## Dans la boucle

```cpp
while (true) {
    uint32_t debut = gb.get_millis();

    Keys k; read_input(k);
    paddle_update(paddle, k);
    ball_update(ball);                 // <-- la balle avance et rebondit

    gfx.clear(color_black);
    paddle_draw(paddle);
    gfx.setColor(color_yellow);
    gfx.fillRect((int)ball.x, (int)ball.y, ball.size, ball.size);
    gfx.update();

    uint32_t travail = gb.get_millis() - debut;
    if (travail < FRAME_MS) gb.delay_ms(FRAME_MS - travail);
}
```

**À tester :** la balle part en diagonale et rebondit proprement sur les trois murs
(gauche, droite, plafond), à vitesse constante. Elle traverse encore la raquette : c'est
justement le sujet du chapitre 10.

---

## À retenir

- Une balle = **position + vitesse** ; avancer = **position += vitesse** chaque image.
- On calcule en **`float`** et on **arrondit `(int)`** seulement pour dessiner.
- Rebond sur un mur = **inverser** la vitesse concernée, après avoir **replacé** la balle
  sur le bord.

---

[« Précédent](Chapitre_08.md) | [Accueil](index.md) | [Suivant » : Collision Arkanoid](Chapitre_10.md)
