# Chapitre 21 — Assemblage final et publication

[« Précédent](Chapitre_20.md) | [Accueil](index.md) | [Suivant »](Chapitre_bonus.md)

![Niveau](https://img.shields.io/badge/Niveau-Avanc%C3%A9-red)

---

## Objectif

Rassembler tout ce qu'on a construit en un projet **propre**, découpé en fichiers, et le
publier sur GitHub.

---

## Découper le code en fichiers

Jusqu'ici on a pu tout garder dans `app_main.cpp` pour apprendre. Pour un vrai projet, on
sépare par **responsabilité** — un couple `.h`/`.cpp` par domaine :

```
main/
├── CMakeLists.txt
├── app_main.cpp        ← init + boucle + machine à états
├── config.h            ← constantes (tailles, vitesses, couleurs)
├── entities.h          ← Rect, Ball, Paddle, Brick, structures communes
├── input.h / .cpp      ← Keys + read_input (chapitre 7)
├── physics.h / .cpp    ← overlap, rebonds, collisions (chapitres 9-11)
├── level.h / .cpp      ← plans + génération (chapitres 15-16)
├── audio.h / .cpp      ← player, voices, SfxBus (chapitre 13)
├── save.h / .cpp       ← lecture/écriture SD (chapitre 17)
├── i18n.h / .cpp       ← table de textes (chapitre 18)
└── ui.h / .cpp         ← menus (chapitre 19)
```

Le **`.h`** (en-tête) déclare *ce que le module offre* (les fonctions, les structures) ;
le **`.cpp`** contient *le code*. Les autres fichiers font `#include "physics.h"` pour
utiliser la physique sans connaître ses détails. C'est le développement « en couches »
appliqué à l'échelle du projet.

> Rappel du chapitre 3 : chaque nouveau `.cpp` doit être ajouté au `SRCS` du
> `main/CMakeLists.txt`, sinon il n'est pas compilé.

---

## Le squelette de `app_main.cpp`

Tout se ramène à : initialiser, charger, puis tourner la boucle qui distribue le travail
selon l'état.

```cpp
#include "gamebuino.h"
#include "config.h"
#include "input.h"
#include "audio.h"
#include "save.h"
#include "i18n.h"
// ...

gb_core     gb;
gb_graphics gfx;

extern "C" void app_main(void) {
    gb.init();
    audio_init();                 // tâche + volume (chapitre 13)
    SaveData save; save_read(save);
    set_lang(save.lang[0]=='E' ? EN : FR);
    player.set_master_volume(save.volume);

    Game g; new_game(g);

    while (true) {
        uint32_t debut = gb.get_millis();
        Keys k; read_input(k);

        checkReturnToLoader(k);   // RUN+MENU 500 ms (voir ci-dessous)

        switch (g.state) {
            case State::Title:       update_title(g, k);   break;
            case State::WaitingBall: update_waiting(g, k); break;
            case State::Playing:     update_playing(g, k); break;
            case State::Pause:       update_pause(g, k);   break;
            case State::Options:     update_options(g, k, save); break;
            case State::GameOver:    update_gameover(g, k);break;
        }

        gfx.update();
        uint32_t travail = gb.get_millis() - debut;
        if (travail < FRAME_MS) gb.delay_ms(FRAME_MS - travail);
    }
}
```

---

## Revenir au menu de la console (loader)

Un jeu propre sait rendre la main. On revient au **loader** de la AKA en redémarrant sur
sa partition, quand le joueur maintient **RUN + MENU** ~500 ms — une combinaison peu
probable par accident.

```cpp
#include "esp_ota_ops.h"
#include "esp_partition.h"

void checkReturnToLoader(const Keys& k) {
    static uint32_t combo_start = 0;
    uint32_t now = gb.get_millis();
    bool run_menu = k.run && k.menu;             // les deux maintenues

    if (run_menu) {
        if (combo_start == 0) combo_start = now;
        else if (now - combo_start >= 500) {
            const esp_partition_t* loader = esp_partition_find_first(
                ESP_PARTITION_TYPE_APP, ESP_PARTITION_SUBTYPE_APP_OTA_1, nullptr);
            if (loader) { esp_ota_set_boot_partition(loader); esp_restart(); }
        }
    } else combo_start = 0;                       // relâché : on réarme
}
```

> Appelle-la **à chaque image, dans tous les états** (pas seulement en jeu), à partir de
> la lecture unique `read_input`. `k.run` / `k.menu` viennent de `gb.buttons.state()` avec
> `KEY_RUN` / `KEY_MENU`.

---

## Publier sur GitHub

Une fois le jeu prêt, un dépôt soigné donne envie de l'essayer :

- un **README** clair : de quoi il s'agit, une **capture d'écran** (ou un GIF), comment
  compiler (`idf.py set-target esp32s3` → `build` → `flash`), les commandes du jeu ;
- la **licence** (par ex. MIT) ;
- le dossier `components/gamebuino` **tel quel** (ou un lien vers la lib + instructions
  pour la déposer) ;
- éventuellement un dossier `assets/` (sprites, sons) et un `docs/` (ce tutoriel !).

**À tester :** clone ton propre dépôt dans un dossier neuf, compile, flashe. Si ça marche
« à froid », il est prêt à être partagé.

---

## À retenir

- Découpe par **responsabilité** (`.h` = ce qu'on offre, `.cpp` = le code) et **ajoute
  chaque `.cpp`** au `CMakeLists.txt`.
- `app_main` = init + chargement + boucle qui **distribue** selon l'état.
- Prévois le **retour au loader** (RUN+MENU) et un **README** de qualité pour publier.

---

[« Précédent](Chapitre_20.md) | [Accueil](index.md) | [Suivant » : Bonus](Chapitre_bonus.md)
