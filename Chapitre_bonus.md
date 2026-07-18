# Chapitre Bonus — Effets : particules, sprites animés, plein écran, boss

[« Précédent](Chapitre_21.md) | [Accueil](index.md)

![Niveau](https://img.shields.io/badge/Niveau-Expert-purple)

---

## Objectif

Ajouter du « jus » : des petits effets qui donnent vie au jeu. Tous reposent sur des
notions déjà vues (structures, `vector`, framebuffer, blit). Ce chapitre est une boîte à
idées avec du code de départ — à picorer selon l'envie.

---

## Particules : un `vector` de petits points qui vivent puis meurent

Une particule est un point avec une **position**, une **vitesse**, une **durée de vie**
et une couleur. À chaque image elle avance, vieillit, puis disparaît. C'est exactement la
même mécanique que la balle (chapitre 9), en tout petit et en grand nombre.

```cpp
struct Particle { float x, y, vx, vy; int life; uint16_t col; };
std::vector<Particle> particles;

// à la casse d'une brique : projeter quelques éclats
void spawn_burst(int x, int y, uint16_t col) {
    for (int n = 0; n < 8; n++) {
        float a = (rand() % 628) / 100.0f;         // angle 0..2π (approx)
        float s = 1.0f + (rand() % 200) / 100.0f;  // vitesse
        particles.push_back({ (float)x, (float)y,
                              cosf(a)*s, sinf(a)*s, 20, col });   // vit 20 images
    }
}

void particles_update() {
    for (auto& p : particles) {
        if (p.life <= 0) continue;
        p.x += p.vx; p.y += p.vy;
        p.vy += 0.05f;          // un peu de gravité : les éclats retombent
        p.life--;               // vieillit
    }
}

void particles_draw() {
    for (auto& p : particles)
        if (p.life > 0) gfx.drawPixel((int)p.x, (int)p.y, p.col);
}
```

Comme pour les briques, on **réutilise** les particules « mortes » (`life <= 0`) plutôt
que de les supprimer du `vector` en plein jeu (chapitre 11). Pense à `particles.reserve(...)`
(chapitre 20).

---

## Sprites animés : changer d'image au fil du temps

Un sprite animé, c'est plusieurs images (*frames*) qu'on affiche à tour de rôle. On garde
un compteur : toutes les N images de jeu, on passe à la frame suivante. L'affichage
réutilise le **blit** du chapitre 4.

```cpp
struct AnimSprite {
    const uint16_t* frames[4];   // 4 images
    int w, h;
    int frame = 0;               // image courante
    int timer = 0;               // compteur
    int speed = 6;               // change de frame toutes les 6 images de jeu
};

void anim_update(AnimSprite& a) {
    if (++a.timer >= a.speed) { a.timer = 0; a.frame = (a.frame + 1) % 4; }
}

void anim_draw(const AnimSprite& a, int x, int y) {
    draw_sprite(a.frames[a.frame], a.w, a.h, x, y);   // draw_sprite : chapitre 4
}
```

```
 frame 0   frame 1   frame 2   frame 3   → puis on reboucle
  ▲ ▲       ▲ ▲       ▲ ▲       ▲ ▲
  ███       ███       ███       ███
  █ █       ▐ ▌       █ █       ▐ ▌
```

---

## Effets « plein écran » : écrire directement le framebuffer

Un « flash » blanc, un fondu, une teinte… ce sont des effets qu'on applique à **tout**
l'écran. Comme le framebuffer est un simple tableau (chapitre 4), on peut le parcourir.

```cpp
// flash : recouvre tout l'écran d'une couleur, opacité décroissante gérée par la durée
void flash(uint16_t col) {
    for (int i = 0; i < SCREEN_WIDTH * SCREEN_HEIGHT; i++)
        framebuffer[i] = col;
}
```

Pour un vrai fondu progressif, on mélangerait chaque pixel avec la couleur cible (en
décomposant R/V/B — chapitre 4). Reste **léger** : parcourir 76 800 pixels par image a un
coût, à réserver aux moments clés (transition, dégâts, victoire).

---

## Un boss : une grosse entité avec de la vie et un comportement

Un boss n'est qu'un objet plus gros, avec des **points de vie** et un **déplacement**
scripté. On réutilise `Rect`, la collision `overlap`, et un petit automate pour son
comportement (va-et-vient, phases selon les hp…).

```cpp
struct Boss {
    Rect r;
    int  hp;
    float vx;      // vitesse de patrouille
};

void boss_update(Boss& b) {
    b.r.x += (int)b.vx;
    if (b.r.x < 0 || b.r.x > SCREEN_W - b.r.w) b.vx = -b.vx;   // rebond sur les bords
    // touché par la balle ? -> overlap(ball_rect(ball), b.r) -> b.hp--
    // hp bas -> change de phase (va plus vite, tire...)
}
```

---

## À retenir

- Tous ces effets **réutilisent** l'acquis : `vector` (particules), blit (sprites
  animés), framebuffer (plein écran), `Rect`/`overlap` (boss).
- On **recycle** les objets morts plutôt que de les supprimer en plein jeu, et on
  **réserve** la mémoire à l'avance.
- Les effets plein écran sont puissants mais **coûteux** : à utiliser aux moments forts.

Bravo — tu as fait le tour de tout ce qui compose un vrai petit moteur de jeu sur la AKA.
À toi de créer le tien. 🎮

---

[« Précédent](Chapitre_21.md) | [Accueil](index.md)
