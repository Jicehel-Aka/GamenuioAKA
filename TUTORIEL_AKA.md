# Créer un jeu sur Gamebuino AKA — tutoriel pas à pas

> Un guide progressif pour écrire un jeu **from scratch** sur la **Gamebuino AKA**
> (ESP32‑S3, écran 320×240, ESP‑IDF, C++), en ajoutant les briques une par une.
> Chaque étape est **courte** et se termine par un **programme testable sur la console**.

À la fin, tu auras construit un petit casse‑briques et surtout compris **tous les
systèmes** qu'on réutilise d'un jeu à l'autre : boucle de jeu, entrées, rendu,
physique, collisions, machine à états, audio, bonus, génération de niveaux,
sauvegarde sur carte SD, multilingue et capture d'écran.

---

## Sommaire

1. [Prérequis et structure du projet](#etape-0)
2. [Premier boot : initialiser la console](#etape-1)
3. [Dessiner : pixels, couleurs, texte](#etape-2)
4. [La boucle de jeu et le pas de temps fixe](#etape-3)
5. [Les entrées : boutons et joystick](#etape-4)
6. [Une raquette qui bouge](#etape-5)
7. [Une balle qui rebondit sur les murs](#etape-6)
8. [Collision balle / raquette (angle façon Arkanoid)](#etape-7)
9. [Les briques : grille, collision, casse](#etape-8)
10. [La machine à états du jeu](#etape-9)
11. [Le son : tons, tâche audio, mixage](#etape-10)
12. [Les bonus qui tombent (colle, laser)](#etape-11)
13. [Générer des niveaux + briques incassables](#etape-12)
14. [Sauvegarder sur la carte SD](#etape-13)
15. [Le multilingue (i18n)](#etape-14)
16. [Capture d'écran en BMP](#etape-15)
17. [Retour au loader](#etape-16)
18. [Pour aller plus loin](#pour-aller-plus-loin)

---

<a name="etape-0"></a>
## Étape 0 — Prérequis et structure du projet

### Ce qu'il te faut

- **ESP‑IDF v5.5+** installé et fonctionnel (`idf.py --version`).
- **VS Code** avec l'extension ESP‑IDF (ou la ligne de commande).
- La console **Gamebuino AKA** et son câble USB.
- Le composant **`gamebuino`** (le HAL de la console) : le dossier `components/gamebuino/`
  qui fournit `gb_core`, `gb_graphics` et la couche bas niveau `gb_ll_*`
  (écran, boutons, joystick, audio, carte SD).

### Structure minimale

```
mon_jeu/
├── CMakeLists.txt              ← décrit le PROJET entier
├── sdkconfig                   ← configuration (généré par idf.py, ne pas éditer à la main)
├── components/
│   └── gamebuino/              ← le HAL (fourni) = une bibliothèque réutilisable
└── main/
    ├── CMakeLists.txt          ← décrit le composant "main" (notre code)
    └── app_main.cpp            ← notre code
```

### D'abord : c'est quoi CMake, et pourquoi deux `CMakeLists.txt` ?

**CMake** est l'outil qui **génère la recette de compilation** : quels fichiers
compiler, dans quel ordre, en liant quelles bibliothèques. On ne compile jamais « à la
main » avec `g++` sur ce genre de projet — c'est bien trop de fichiers. On **décrit**
le projet dans des fichiers nommés `CMakeLists.txt`, et l'outil s'occupe du reste.

ESP‑IDF ajoute sa propre couche par‑dessus CMake : le projet est découpé en
**composants** (des mini‑bibliothèques indépendantes). Chaque composant a **son
propre** `CMakeLists.txt`. D'où les deux fichiers :

- celui de la **racine** décrit le **projet** dans son ensemble ;
- celui de **`main/`** décrit **un composant** (ici, ton code de jeu).

Le dossier `components/gamebuino/` est lui aussi un composant (avec son propre
`CMakeLists.txt`, déjà fourni). C'est comme ça qu'on réutilise le HAL sans recopier son
code.

> 📖 Pour approfondir : [documentation officielle CMake](https://cmake.org/documentation/)
> et surtout le [système de build ESP‑IDF](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/build-system.html)
> (la notion de composant, `idf_component_register`, `REQUIRES`, etc.).

### Le `CMakeLists.txt` de la racine (le projet)

```cmake
cmake_minimum_required(VERSION 3.16)                       # (1)
include($ENV{IDF_PATH}/tools/cmake/project.cmake)          # (2)
project(mon_jeu)                                           # (3)
```

Ligne par ligne :

1. **`cmake_minimum_required(VERSION 3.16)`** — refuse de continuer si la version de
   CMake installée est trop ancienne (garantit que la syntaxe utilisée est comprise).
2. **`include(...project.cmake)`** — charge toute la « magie » ESP‑IDF (les fonctions
   comme `idf_component_register`, la gestion des composants, la config `sdkconfig`…).
   `$ENV{IDF_PATH}` pointe vers ton installation ESP‑IDF (défini par `export.sh`).
3. **`project(mon_jeu)`** — déclare le projet et son **nom** (c'est aussi le nom du
   binaire produit, `mon_jeu.bin`). Cette ligne doit venir **après** l'`include`.

### Le `CMakeLists.txt` de `main/` (notre composant)

```cmake
idf_component_register(                                    # (1)
    SRCS "app_main.cpp"                                    # (2)
    INCLUDE_DIRS "."                                       # (3)
    REQUIRES gamebuino                                     # (4)
)
```

1. **`idf_component_register(...)`** — la fonction ESP‑IDF qui **déclare un composant**.
   Tout ce qui suit configure ce composant.
2. **`SRCS`** — la **liste des fichiers à compiler**. Ici un seul (`app_main.cpp`) ;
   plus tard on en ajoutera (`game.cpp`, `audio.cpp`…), séparés par des espaces ou des
   retours à la ligne.
3. **`INCLUDE_DIRS`** — les dossiers où le compilateur cherche les **en‑têtes** (`.h`)
   de **ce** composant. `"."` = le dossier courant (`main/`), pour que
   `#include "mon_fichier.h"` fonctionne.
4. **`REQUIRES`** — la liste des **autres composants dont on dépend**. En mettant
   `gamebuino` ici, notre code peut faire `#include "gb_core.h"` **et** être **lié** à
   la bibliothèque du HAL.

> ⚠️ **Piège classique** : `gamebuino` va dans **`REQUIRES`**, pas dans `INCLUDE_DIRS`.
> `INCLUDE_DIRS` ne sert qu'à exposer les en‑têtes de *ton* composant. Mettre une
> dépendance au mauvais endroit compile parfois (les `.h` sont trouvés) mais **échoue à
> l'édition de liens** (`undefined reference to ...`), car la bibliothèque n'est pas liée.

### Compiler et flasher

```bash
idf.py set-target esp32s3
idf.py build
idf.py -p COM3 flash monitor      # remplace COM3 par ton port
```

**À tester :** la compilation passe. On n'a pas encore de code utile — on s'en occupe
à l'étape suivante.

---

<a name="etape-1"></a>
## Étape 1 — Premier boot : initialiser la console

Tout part d'un objet **`gb_core`**. Son `init()` met en route l'écran, les boutons,
le joystick, l'audio et monte la carte SD. On l'appelle **une seule fois**.

```cpp
// app_main.cpp
#include "gb_core.h"
#include "gb_ll_lcd.h"          // lcd_clear, lcd_refresh, couleurs
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

gb_core g_core;                 // l'objet console, global

extern "C" void app_main(void)
{
    g_core.init();              // écran + entrées + audio + carte SD

    lcd_clear(color_black);     // efface le tampon d'affichage
    lcd_refresh();              // envoie le tampon à l'écran

    while (true) {
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```

Deux fonctions clés reviendront **partout** :

- `lcd_clear(couleur)` : remplit le **framebuffer** (le tampon en mémoire).
- `lcd_refresh()` : **envoie** ce tampon à l'écran physique (transfert DMA).

On dessine toujours **dans le tampon**, puis on **rafraîchit une fois**.

**À tester :** l'écran devient noir et le reste. La console ne plante pas. 🎉

---

<a name="etape-2"></a>
## Étape 2 — Dessiner : pixels, couleurs, texte

### Le format des couleurs (important)

L'écran utilise du **16 bits par pixel**. La fonction `lcd_color_rgb(r, g, b)` (valeurs
0–255) fabrique la couleur. Sur la AKA, le **rouge est dans les bits de poids faible** :

```
bits 0‑4   = rouge (5 bits)
bits 5‑10  = vert  (6 bits)
bits 11‑15 = bleu  (5 bits)
```

Il existe des couleurs prédéfinies : `color_black`, `color_white`, `color_red`,
`color_green`, `color_yellow`, etc.

### Pixels, rectangles, texte

```cpp
lcd_clear(color_black);

// un pixel
lcd_putpixel(160, 120, color_white);

// un rectangle plein, fait main (utile pour raquette/balle)
for (int y = 100; y < 130; y++)
    for (int x = 140; x < 180; x++)
        lcd_putpixel(x, y, color_red);

// du texte (police ASCII intégrée)
lcd_draw_text(90, 10, "Salut AKA !");

lcd_refresh();
```

> 💡 **La police est en ASCII pur** : pas d'accents (é, è, à s'afficheraient mal).
> Écris tes textes **sans accent** (« terminee », « regler »…). On verra à
> l'étape 14 comment gérer proprement plusieurs langues, toujours sans accent.

Un petit utilitaire pratique qu'on réutilisera :

```cpp
inline void draw_rect(int x, int y, int w, int h, uint16_t col) {
    for (int j = 0; j < h; j++)
        for (int i = 0; i < w; i++)
            lcd_putpixel(x + i, y + j, col);
}
```

**À tester :** un carré rouge au centre, un pixel blanc, un titre en haut.

---

<a name="etape-3"></a>
## Étape 3 — La boucle de jeu et le pas de temps fixe

Un jeu, c'est : **lire les entrées → mettre à jour → dessiner**, en boucle. Le piège :
si on met un simple `vTaskDelay(20)`, la durée d'une image = 20 ms **+** le temps de
dessin (variable). Résultat : la vitesse de la balle **fluctue** selon la charge.

La solution : un **limiteur de cadence** qui vise une période **constante**.

```cpp
#include "esp_timer.h"

extern "C" void app_main(void)
{
    g_core.init();

    while (true) {
        uint32_t frame_start = esp_timer_get_time() / 1000;  // ms

        // 1. lire les entrées   (étape 4)
        // 2. mettre à jour       (étapes 5+)
        // 3. dessiner
        lcd_clear(color_black);
        lcd_draw_text(90, 110, "Boucle de jeu OK");
        lcd_refresh();

        // limiteur : chaque image dure EXACTEMENT ~30 ms (~33 images/s)
        const uint32_t FRAME_MS = 30;
        uint32_t work = (esp_timer_get_time() / 1000) - frame_start;
        if (work < FRAME_MS) vTaskDelay(pdMS_TO_TICKS(FRAME_MS - work));
        else                 vTaskDelay(1);   // jamais de délai négatif
    }
}
```

Ainsi, quel que soit le temps de dessin, la physique avance **du même pas**. Si plus
tard le jeu te paraît trop rapide ou lent, c'est le **seul** chiffre à changer :
`FRAME_MS` plus grand = plus lent.

**À tester :** le texte s'affiche de façon stable, sans scintillement.

---

<a name="etape-4"></a>
## Étape 4 — Les entrées : boutons et joystick

Un **seul** endroit lit le matériel : `g_core.pool()`. Il interroge les boutons **et**
le joystick d'un coup (c'est important : ils partagent le même bus, on ne veut pas
deux lecteurs concurrents).

```cpp
g_core.pool();                                   // lit le matériel

auto& btn = g_core.buttons;
bool a_held  = btn.state()   & EXPANDER_KEY_A;   // A est-il MAINTENU ?
bool a_press = btn.pressed()  & EXPANDER_KEY_A;  // A vient-il d'être ENFONCE ? (front)
bool a_rel   = btn.released() & EXPANDER_KEY_A;  // A vient-il d'être RELACHE ?

int jx = g_core.joystick.get_x();                // -1000 .. +1000
int jy = g_core.joystick.get_y();
```

### Niveau vs front — la distinction qui évite les bugs

- **Niveau** (`state()`) : « la touche est‑elle enfoncée maintenant ? » → pour un
  déplacement continu (bouger la raquette tant qu'on pousse).
- **Front** (`pressed()`) : « la touche vient‑elle d'être enfoncée ? » → pour une
  **action unique** (lancer la balle, valider un menu).

> 🐛 **Exemple vécu** : si on lance la balle sur `state()` (niveau), l'appui A qui a
> *démarré la partie* est encore maintenu et **lance la balle aussitôt**. En lançant
> sur `pressed()` (front), il faut relâcher puis re‑presser : plus de bug.

Pour simplifier, on regroupe tout dans une petite structure :

```cpp
struct Keys {
    bool left, right, a, b;      // niveau (maintenu)
    uint32_t pressed, released;  // masques de fronts
    int jx, jy;
};

void read_input(Keys& k) {
    g_core.pool();
    auto& b = g_core.buttons;
    uint32_t s = b.state();
    k.left  = s & EXPANDER_KEY_LEFT;
    k.right = s & EXPANDER_KEY_RIGHT;
    k.a     = s & EXPANDER_KEY_A;
    k.b     = s & EXPANDER_KEY_B;
    k.pressed  = b.pressed();
    k.released = b.released();
    k.jx = g_core.joystick.get_x();
    k.jy = g_core.joystick.get_y();
}
```

**À tester :** affiche `k.jx`/`k.jy` avec `lcd_draw_text` et bouge le stick ; appuie
sur A et vérifie que `pressed & EXPANDER_KEY_A` ne s'allume qu'**un instant**.

---

<a name="etape-5"></a>
## Étape 5 — Une raquette qui bouge

On introduit nos premières **constantes de jeu** (dans un futur `config.h`) et un
objet raquette simple.

```cpp
constexpr int SCREEN_W = 320, SCREEN_H = 240;
constexpr int PADDLE_W = 48, PADDLE_H = 6;
constexpr int PADDLE_Y = SCREEN_H - 20;
constexpr int PADDLE_SPEED = 5;

struct Paddle { int x = (SCREEN_W - PADDLE_W) / 2; };

void paddle_update(Paddle& p, const Keys& k) {
    // clavier
    if (k.left)  p.x -= PADDLE_SPEED;
    if (k.right) p.x += PADDLE_SPEED;
    // joystick (zone morte à ±200 pour ne pas dériver)
    if (k.jx >  200) p.x += PADDLE_SPEED;
    if (k.jx < -200) p.x -= PADDLE_SPEED;
    // on borne à l'écran
    if (p.x < 0) p.x = 0;
    if (p.x > SCREEN_W - PADDLE_W) p.x = SCREEN_W - PADDLE_W;
}

void paddle_draw(const Paddle& p) {
    draw_rect(p.x, PADDLE_Y, PADDLE_W, PADDLE_H, color_white);
}
```

Dans la boucle : `read_input(k); paddle_update(p, k); … paddle_draw(p);`

**À tester :** la raquette suit le stick et les flèches, et s'arrête aux bords.

---

<a name="etape-6"></a>
## Étape 6 — Une balle qui rebondit sur les murs

La balle a une **position** et une **vitesse** (en flottant, pour des rebonds nets).

```cpp
constexpr int   BALL_SIZE  = 6;
constexpr float BALL_SPEED = 2.6f;

struct Ball { float x, y, vx, vy; bool active = false; };

void ball_update(Ball& b) {
    if (!b.active) return;
    b.x += b.vx;
    b.y += b.vy;

    // murs gauche/droite
    if (b.x < 0)                    { b.x = 0;                    b.vx = -b.vx; }
    if (b.x > SCREEN_W - BALL_SIZE) { b.x = SCREEN_W - BALL_SIZE; b.vx = -b.vx; }
    // plafond
    if (b.y < 0)                    { b.y = 0;                    b.vy = -b.vy; }
    // (le bas = perte de balle : on le gère à l'étape 9)
}

void ball_draw(const Ball& b) {
    draw_rect((int)b.x, (int)b.y, BALL_SIZE, BALL_SIZE, color_yellow);
}
```

Pour tester tout de suite, lance la balle au démarrage :

```cpp
Ball ball{ 160, 120, BALL_SPEED, -BALL_SPEED, true };
```

**À tester :** la balle part en diagonale et rebondit sur les trois murs.

---

<a name="etape-7"></a>
## Étape 7 — Collision balle / raquette (angle façon Arkanoid)

Un simple `vy = -vy` suffit à faire rebondir, mais le jeu devient plat. La bonne
recette **Arkanoid** : **l'endroit** où la balle touche la raquette détermine
**l'angle** de renvoi. Bord gauche → part à gauche, centre → tout droit, etc. La
**vitesse reste constante** (on ne fait que tourner le vecteur).

```cpp
#include <cmath>

bool overlap(const Ball& b, const Paddle& p) {
    return b.x < p.x + PADDLE_W && b.x + BALL_SIZE > p.x &&
           b.y + BALL_SIZE > PADDLE_Y && b.y < PADDLE_Y + PADDLE_H;
}

void ball_vs_paddle(Ball& b, const Paddle& p) {
    if (b.vy <= 0 || !overlap(b, p)) return;      // seulement quand elle descend

    // position d'impact : -1 (gauche) .. +1 (droite)
    float hit = ((b.x + BALL_SIZE/2.0f) - (p.x + PADDLE_W/2.0f)) / (PADDLE_W/2.0f);
    if (hit < -1) hit = -1;
    if (hit >  1) hit =  1;

    float angle = hit * 1.05f;                    // ±60° environ
    float speed = std::sqrt(b.vx*b.vx + b.vy*b.vy);
    b.vx =  speed * std::sin(angle);
    b.vy = -speed * std::cos(angle);              // toujours renvoyée vers le haut
    b.y  = PADDLE_Y - BALL_SIZE - 1;              // on la sort de la raquette
}
```

**À tester :** touche la balle avec le bord de la raquette → elle repart en biais ;
avec le centre → presque à la verticale. La vitesse ne change pas.

---

<a name="etape-8"></a>
## Étape 8 — Les briques : grille, collision, casse

Une brique a une position, des **points de vie** (`hp`), un état `alive`.

```cpp
constexpr int BRICK_W = 28, BRICK_H = 14, BRICK_TOP = 28;
constexpr int BRICK_COLS = 9, BRICK_ROWS = 6, BRICK_GAP = 4;

struct Brick { int x, y, hp; bool alive; uint16_t col; };

std::vector<Brick> make_grid(int rows, int hp) {
    std::vector<Brick> v;
    int total = BRICK_COLS * BRICK_W + (BRICK_COLS - 1) * BRICK_GAP;
    int ox = (SCREEN_W - total) / 2;
    uint16_t palette[] = { color_red, color_yellow, color_green, color_white };
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < BRICK_COLS; c++)
            v.push_back({ ox + c*(BRICK_W+BRICK_GAP),
                          BRICK_TOP + r*(BRICK_H+BRICK_GAP),
                          hp, true, palette[r % 4] });
    return v;
}
```

La collision détermine l'**axe de rebond** en comparant le chevauchement horizontal
et vertical (plus robuste que de se fier au seul signe de `vy`) :

```cpp
void ball_vs_bricks(Ball& b, std::vector<Brick>& bricks, int& score) {
    for (auto& k : bricks) {
        if (!k.alive) continue;
        bool hit = b.x < k.x + BRICK_W && b.x + BALL_SIZE > k.x &&
                   b.y < k.y + BRICK_H && b.y + BALL_SIZE > k.y;
        if (!hit) continue;

        if (--k.hp <= 0) { k.alive = false; score += 100; }

        // axe de rebond selon la pénétration la plus faible
        float dx = (b.x + BALL_SIZE/2.0f) - (k.x + BRICK_W/2.0f);
        float dy = (b.y + BALL_SIZE/2.0f) - (k.y + BRICK_H/2.0f);
        float ox = (BRICK_W + BALL_SIZE)/2.0f - std::fabs(dx);
        float oy = (BRICK_H + BALL_SIZE)/2.0f - std::fabs(dy);
        if (ox < oy) b.vx = -b.vx; else b.vy = -b.vy;
        break;                        // une brique par image suffit
    }
}

void bricks_draw(const std::vector<Brick>& bricks) {
    for (auto& k : bricks)
        if (k.alive) draw_rect(k.x, k.y, BRICK_W, BRICK_H, k.col);
}
```

**À tester :** la balle casse les briques et rebondit correctement dessus. Le score
monte (affiche‑le avec `lcd_draw_text`).

---

<a name="etape-9"></a>
## Étape 9 — La machine à états du jeu

Un jeu n'est jamais « toujours en train de jouer ». On gère des **états** : écran‑titre,
attente du lancement, jeu, game over. Ça structure tout le reste.

```cpp
enum class State { Title, WaitingBall, Playing, GameOver };

struct Game {
    State state = State::Title;
    Paddle paddle;
    std::vector<Ball> balls;
    std::vector<Brick> bricks;
    int score = 0, lives = 3;
};
```

- **Title** : « Appuie sur A ». Sur **front** A → on prépare une partie et on passe en
  `WaitingBall`.
- **WaitingBall** : la balle est **posée sur la raquette** (elle la suit). Sur **front**
  A → on la lance (`Playing`). *(Front, pas niveau : cf. étape 4.)*
- **Playing** : la boucle complète (update balle, collisions). Si la balle passe sous
  l'écran → on perd une vie ; plus de vie → `GameOver`.
- **GameOver** : « Partie terminee, A pour rejouer ».

Extrait de la boucle :

```cpp
switch (g.state) {
case State::Title:
    lcd_draw_text(80, 120, "Appuie sur A pour jouer");
    if (k.pressed & EXPANDER_KEY_A) { new_game(g); g.state = State::WaitingBall; }
    break;

case State::WaitingBall:
    // la balle suit la raquette
    g.balls[0].x = g.paddle.x + PADDLE_W/2 - BALL_SIZE/2;
    g.balls[0].y = PADDLE_Y - BALL_SIZE;
    draw_all(g);
    lcd_draw_text(90, 150, "A pour lancer");
    if (k.pressed & EXPANDER_KEY_A) {
        g.balls[0].vx = BALL_SPEED; g.balls[0].vy = -BALL_SPEED;
        g.balls[0].active = true;   g.state = State::Playing;
    }
    break;

case State::Playing:
    paddle_update(g.paddle, k);
    for (auto& b : g.balls) { ball_update(b); ball_vs_paddle(b, g.paddle);
                              ball_vs_bricks(b, g.bricks, g.score); }
    // perte de balle
    if (g.balls[0].y > SCREEN_H) {
        if (--g.lives <= 0) g.state = State::GameOver;
        else                g.state = State::WaitingBall;
    }
    draw_all(g);
    break;

case State::GameOver:
    lcd_draw_text(70, 120, "Partie terminee - A pour rejouer");
    if (k.pressed & EXPANDER_KEY_A) g.state = State::Title;
    break;
}
```

> 💡 **Unifie le démarrage** : fais passer **toutes** les mises en jeu (nouvelle
> partie, nouvelle vie, niveau suivant) par l'état `WaitingBall`. Tu évites d'avoir
> deux chemins de lancement différents (source de bugs subtils).

**À tester :** cycle complet titre → jeu → game over → titre. On lance la balle avec A
seulement après l'avoir relâchée.

---

<a name="etape-10"></a>
## Étape 10 — Le son : tons, tâche audio, mixage

Le son se joue avec des **pistes de ton** (`gb_audio_track_tone`) branchées sur un
**lecteur** (`gb_audio_player`). Deux points cruciaux tirés de l'expérience :

### 1) Il faut « pooler » très souvent — dans une tâche dédiée

Le mixeur doit être **alimenté toutes les ~2 ms**, sinon le tampon audio se vide et
les sons deviennent quasi inaudibles. On lui dédie donc une **tâche FreeRTOS**, seule
appelante de `pool()` :

```cpp
#include "gb_audio_player.h"
#include "gb_audio_track_tone.h"
#include "gb_ll_audio.h"

gb_audio_player       player;
gb_audio_track_tone   voices[4];      // 4 voix (le lecteur en gère 4)

static void audio_task(void*) {
    while (true) { player.pool(); vTaskDelay(pdMS_TO_TICKS(2)); }
}

void audio_init() {
    gb_ll_audio_set_volume(200);      // volume MATERIEL du codec (à ne pas oublier)
    for (auto& v : voices) { player.add_track(&v); v.set_track_volume(1.0f); }
    xTaskCreatePinnedToCore(audio_task, "AudioTask", 4096, nullptr, 5, nullptr, 0);
}
```

> ⚠️ Le lecteur n'a que **4 voix** (`AUDIO_PLAYER_TRACK_COUNT`). Enregistrer plus de
> pistes que ça : les suivantes sont **ignorées silencieusement**.

### 2) Un bus SFX en « round‑robin » + un gain par effet

Comme on n'a pas de musique, on met les **4 voix** au service des effets, en rotation :
chaque son part sur la voix suivante, donc un son n'en coupe pas un autre.

```cpp
struct SfxBus {
    int next = 0;
    void play(float freq, uint16_t ms, float gain = 1.0f) {
        voices[next].play_tone(freq, gain, ms);
        next = (next + 1) % 4;
    }
};
SfxBus sfx;
```

Le **gain par effet** sert à équilibrer : un son grave (ex. 220 Hz) est **perçu plus
faible** qu'un aigu à volume égal. On monte les sons importants et on baisse les
petits bruits d'interface :

```cpp
sfx.play(300, 150, 1.0f);   // casse de brique : présente
sfx.play(523, 140, 0.55f);  // rebond raquette : discret
sfx.play(660,  35, 0.5f);   // bip de menu : léger
```

**À tester :** casse une brique → « ploc » net ; rebonds discrets ; aucun son ne se
fait rare ni haché.

---

<a name="etape-11"></a>
## Étape 11 — Les bonus qui tombent (colle, laser)

Quand une brique casse, elle peut lâcher un **bonus** qui tombe ; s'il touche la
raquette, il s'active. On stocke les bonus qui tombent, on les fait descendre, et on
teste la collision avec la raquette.

```cpp
enum class Power { Glue, Laser, Multi };
struct Falling { float x, y; Power type; bool active; };

std::vector<Falling> falling;

// à la casse d'une brique (1 chance sur 6) :
if (rand() % 6 == 0)
    falling.push_back({ (float)k.x, (float)k.y, Power::Glue, true });

// chaque image :
for (auto& f : falling) {
    if (!f.active) continue;
    f.y += 2;
    if (/* f touche la raquette */) { apply_power(g, f.type); f.active = false; }
    if (f.y > SCREEN_H) f.active = false;
}
```

### La colle (glue), et son piège d'états

La **colle** attrape la balle sur la raquette pendant une durée (un `sticky_timer`).
Ce même mécanisme sert aussi à **poser** la balle avant le lancement initial. D'où un
piège : il faut distinguer les deux, sinon la balle **colle pour toujours**.

Règle simple qui marche :

- `sticky_timer == -1` → **pose initiale** (avant lancement). Au lancement, on
  **retire** la colle.
- `sticky_timer > 0` → **bonus colle actif**, il décompte. Quand il tombe à 0 : on
  **relâche** la balle capturée et on **retire** la colle.
- Le sprite « collant » ne s'affiche que pour le **vrai bonus** (`sticky_timer > 0`),
  pas pour la pose initiale.

> 🐛 Le bug d'origine : à l'expiration, l'ancien code remettait `sticky_timer = -1`
> **en gardant** le flag colle → la balle restait collée indéfiniment. La correction :
> à l'expiration, **relâcher + retirer** la colle proprement.

**À tester :** casse des briques jusqu'à obtenir la colle ; la balle est capturée puis
relâchée quand le bonus expire ; ensuite elle rebondit normalement.

---

<a name="etape-12"></a>
## Étape 12 — Générer des niveaux + briques incassables

Régénérer toujours la même grille est vite lassant. On écrit un générateur qui varie
les **motifs** et la **difficulté** selon le numéro de niveau, et introduit des
**briques incassables**.

```cpp
std::vector<Brick> generate_level(int index) {
    std::vector<Brick> v;
    int rows   = 3 + (index % 4);          // 3..6 rangées
    int baseHp = 1 + (index / 3);          // +1 hp tous les 3 niveaux
    int motif  = index % 5;                // 5 motifs cycliques
    bool indes = (index >= 3);             // incassables dès le niveau 4
    // ... placement selon le motif (plein, damier, pyramide, colonnes, cadre) ...
    // ... hp de la rangée du haut +1, quelques briques indestructibles ...
    return v;
}
```

Trois règles à ne **pas** oublier pour les briques incassables :

1. **Collision balle** : elles **rebondissent sans dégât ni casse** (petit « clink »).
2. **Fin de niveau** : le test « reste‑t‑il des briques ? » doit **ignorer** les
   incassables — `alive && !indestructible` — sinon le niveau ne se termine **jamais**.
3. **Laser** : il ne les détruit pas non plus ; le tir s'arrête dessus.

Et une **sécurité** : ne jamais générer un niveau composé **uniquement** d'incassables
(sinon impossible à finir) → on ajoute au moins une rangée destructible.

**À tester :** chaque niveau change de motif et monte en difficulté ; les briques
grises renvoient la balle sans casser ; le niveau se termine quand même.

---

<a name="etape-13"></a>
## Étape 13 — Sauvegarder sur la carte SD

La carte SD est déjà **montée** par `g_core.init()` sur `/sdcard`. **N'appelle pas** le
montage une deuxième fois : un second `esp_vfs_fat_sdmmc_mount` sous une carte déjà
montée **casse l'accès** aux fichiers (bug vécu : « aucun score », sauvegarde
impossible).

### Noms de fichiers : le 8.3

Le système de fichiers est FAT. Pour éviter tout souci, garde des **noms 8.3** stricts
(8 caractères max, extension 3) : `SCORES.DAT`, `SETTINGS.DAT`… dans un dossier court.
*(Les noms longs marchent si le « LFN » est activé dans `sdkconfig`, mais le 8.3 est le
choix sûr et portable.)*

### Écrire / lire un fichier binaire

```cpp
#include <cstdio>
struct Score { char name[9]; int32_t points; };   // 16 octets (avec alignement)

void save_scores(const std::vector<Score>& v) {
    FILE* f = fopen("/sdcard/AKBRICKS/SCORES.DAT", "wb");
    if (!f) { printf("SD: echec ecriture\n"); return; }
    fwrite(v.data(), sizeof(Score), v.size(), f);
    fflush(f); fclose(f);
}

std::vector<Score> load_scores() {
    std::vector<Score> v;
    FILE* f = fopen("/sdcard/AKBRICKS/SCORES.DAT", "rb");
    if (!f) return v;                        // pas de fichier = pas de score
    fseek(f, 0, SEEK_END); long n = ftell(f) / sizeof(Score); fseek(f, 0, SEEK_SET);
    v.resize(n); fread(v.data(), sizeof(Score), n, f); fclose(f);
    return v;
}
```

> ⚠️ **Pas d'horloge** : l'ESP32 n'a pas de RTC réglée, donc les fichiers n'ont pas de
> date de modification. C'est **normal**, pas un bug.

> 💡 Range **réglages** (volume, langue) et **scores** au même endroit, dans un petit
> fichier binaire avec un **octet magique** en tête pour ne pas relire un fichier
> étranger.

**À tester :** enregistre un score, redémarre la console, le score est toujours là.

---

<a name="etape-14"></a>
## Étape 14 — Le multilingue (i18n)

On centralise les textes dans une table `[langue][chaîne]` et une fonction `T()` qui
renvoie la bonne version. **Toujours sans accent** (police ASCII).

```cpp
enum Lang { FR, EN, LANG_COUNT };
enum Str  { STR_PLAY, STR_PAUSE, STR_GAMEOVER, STR_COUNT };

static const char* K[LANG_COUNT][STR_COUNT] = {
    { "Jouer", "Pause", "Partie terminee" },   // FR
    { "Play",  "Pause", "Game over"       },   // EN
};

static Lang s_lang = FR;
const char* T(Str s)        { return K[s_lang][s]; }
void set_language(Lang l)   { s_lang = l; }
```

On persiste la langue **sur la carte SD** (cf. étape 13), pas ailleurs, pour rester
cohérent avec le reste des réglages.

> ✅ **Astuce anti‑bug** : écris un petit script (ou un test) qui vérifie que chaque
> langue a **exactement** `STR_COUNT` chaînes. Une colonne manquante décale toute la
> table.

**À tester :** un réglage « Langue » dans ton menu bascule les textes ; le choix
survit à un redémarrage.

---

<a name="etape-15"></a>
## Étape 15 — Capture d'écran en BMP

Sympa pour documenter le jeu : sauver le contenu de l'écran en **BMP** sur la carte SD.
On lit chaque pixel du framebuffer et on convertit vers le format BMP (octets **B, G, R**).

```cpp
// pour un pixel 16 bits de l'écran (rouge en bits 0-4) :
uint16_t v = lcd_getpixel(x, y);
uint8_t r = ( v        & 0x1F) * 255 / 31;   // bits 0-4  = rouge
uint8_t g = ((v >> 5)  & 0x3F) * 255 / 63;   // bits 5-10 = vert
uint8_t b = ((v >> 11) & 0x1F) * 255 / 31;   // bits 11-15= bleu
// dans le fichier BMP on écrit dans l'ordre : b, g, r
```

Vérification rapide de la conversion : un pixel **rouge pur** vaut `0x001F` → donne
`R = 255` en BMP ; un **bleu pur** `0xF800` → `B = 255`. ✔️

> 💡 Déclenche la capture sur un appui **long** de MENU (par ex. 500 ms), pour ne pas
> la confondre avec l'ouverture d'un menu sur appui court. Affiche un court message
> « Capture enregistree » **après** avoir pris l'image (pour qu'il ne soit pas dans le
> BMP).

**À tester :** appui long MENU → un fichier `SHOT0001.BMP` apparaît sur la carte,
ouvrable sur PC avec les bonnes couleurs.

---

<a name="etape-16"></a>
## Étape 16 — Retour au loader

Pour revenir au menu de la console (le « loader ») sans éteindre, on redémarre sur la
partition OTA correspondante :

```cpp
#include "esp_ota_ops.h"
#include "esp_partition.h"

void return_to_loader() {
    const esp_partition_t* loader = esp_partition_find_first(
        ESP_PARTITION_TYPE_APP, ESP_PARTITION_SUBTYPE_APP_OTA_1, nullptr);
    if (loader) { esp_ota_set_boot_partition(loader); esp_restart(); }
}
```

> 💡 Déclenche‑le sur une **combinaison** peu probable par accident, par exemple
> **RUN + MENU maintenus 500 ms**, pour éviter les sorties involontaires.

**À tester :** la combinaison choisie ramène bien à l'écran d'accueil de la console.

---

<a name="pour-aller-plus-loin"></a>
## Pour aller plus loin

Tu as maintenant l'ossature réutilisable pour **tous** tes jeux AKA. Quelques
extensions naturelles :

- **Sprites** : au lieu de rectangles pleins, dessine des images (transparence par
  couleur‑clé, ex. magenta `0xF81F`, ignorée au tracé).
- **Éditeur de niveaux** intégré, qui écrit/relit tes niveaux sur la carte SD.
- **Tâches séparées** : garder un **seul propriétaire** du matériel (une tâche qui lit
  I2C/ADC), l'audio sur sa tâche, l'IA/jeu sur la sienne — sur des cœurs différents.
- **IA** (pour un Pong, un jeu de plateau…) : la faire tourner sur le **CPU 1** pour ne
  pas bousculer l'audio du **CPU 0**.

Bon code, et amuse‑toi bien sur la AKA ! 🎮
