# Chapitre 03 — Structure du projet

[« Précédent](Chapitre_02.md) | [Accueil](index.md) | [Suivant »](Chapitre_04.md)

![Niveau](https://img.shields.io/badge/Niveau-D%C3%A9butant-green)

---

## Objectif

Comprendre **où** va ton code, **pourquoi** il y a des fichiers `CMakeLists.txt`, et
mettre en place un projet vide qui compile.

---

## L'arborescence minimale

```
mon_jeu/
├── CMakeLists.txt              ← décrit le PROJET entier
├── sdkconfig                   ← configuration (généré par idf.py, ne pas éditer à la main)
├── components/
│   └── gamebuino/              ← la bibliothèque AKA (fournie) — on n'y touche PAS
└── main/
    ├── CMakeLists.txt          ← décrit le composant "main" (ton code)
    └── app_main.cpp            ← ton point d'entrée
```

Deux dossiers comptent :

- **`main/`** : c'est **ton** code. Tu y ajouteras `game.cpp`, `paddle.cpp`, etc.
- **`components/gamebuino/`** : la **bibliothèque AKA**. Tu la déposes ici telle quelle.
  C'est une brique indépendante, maintenue à part ; tu ne la modifies pas (si tu trouves
  un bug ou veux une évolution, ça se signale à l'auteur, ça ne se bricole pas en local).

---

## C'est quoi CMake, et pourquoi deux `CMakeLists.txt` ?

Un vrai projet, c'est des dizaines de fichiers `.cpp`/`.h`. On ne les compile pas à la
main un par un. On **décrit** le projet, et un outil génère la « recette » de
compilation. Cet outil, c'est **CMake**.

ESP-IDF ajoute une notion par-dessus CMake : le projet est découpé en **composants**,
de petites bibliothèques indépendantes. **Chaque composant a son propre
`CMakeLists.txt`.** D'où les deux fichiers :

- celui de la **racine** décrit le **projet** dans son ensemble ;
- celui de **`main/`** décrit **un composant** (ton code de jeu).

`components/gamebuino/` est lui aussi un composant, avec son propre `CMakeLists.txt`
(déjà écrit). C'est ainsi qu'on **réutilise** la lib sans recopier son code.

> 📖 Pour creuser : [documentation CMake](https://cmake.org/documentation/) et surtout
> le [système de build ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/build-system.html)
> (composants, `idf_component_register`, `REQUIRES`).

### Le `CMakeLists.txt` de la racine

```cmake
cmake_minimum_required(VERSION 3.16)                 # (1)
include($ENV{IDF_PATH}/tools/cmake/project.cmake)    # (2)
project(mon_jeu)                                     # (3)
```

1. Refuse de continuer si CMake est trop vieux (garantit que la syntaxe est comprise).
2. Charge la « mécanique » ESP-IDF (les fonctions comme `idf_component_register`, la
   gestion des composants…). `$ENV{IDF_PATH}` pointe vers ton installation ESP-IDF.
3. Déclare le projet et son **nom** (= le nom du binaire produit). À mettre **après**
   l'`include`.

### Le `CMakeLists.txt` de `main/`

```cmake
idf_component_register(              # (1)
    SRCS "app_main.cpp"             # (2)
    INCLUDE_DIRS "."                # (3)
    REQUIRES gamebuino             # (4)
)
```

1. La fonction ESP-IDF qui **déclare un composant**.
2. **`SRCS`** : la liste des fichiers à **compiler**. Ici un seul ; plus tard tu en
   ajouteras (`game.cpp`, `paddle.cpp`…), séparés par des espaces ou des retours ligne.
3. **`INCLUDE_DIRS`** : les dossiers où chercher **tes** en-têtes (`.h`). `"."` = le
   dossier `main/`, pour que `#include "mon_fichier.h"` marche.
4. **`REQUIRES`** : les **autres composants dont tu dépends**. En mettant `gamebuino`
   ici, ton code peut faire `#include "gamebuino.h"` **et** être lié à la lib.

> ⚠️ **LE piège** : `gamebuino` va dans **`REQUIRES`**, jamais dans `INCLUDE_DIRS`.
> `INCLUDE_DIRS` sert seulement à exposer *tes* en-têtes. Une dépendance mal placée
> compile parfois (les `.h` sont trouvés) mais **échoue à l'édition de liens**
> (`undefined reference to ...`), car la bibliothèque n'est pas reliée au programme.

---

## Le fichier `app_main.cpp` (le point d'entrée)

Sur ESP-IDF, tout programme démarre dans une fonction nommée **`app_main`**. Le
`extern "C"` demande au compilateur de ne pas « décorer » ce nom, pour qu'ESP-IDF
puisse la retrouver et l'appeler au démarrage.

```cpp
#include "gamebuino.h"   // toute l'API AKA en un seul include

gb_core     gb;          // l'objet "console" : écran, boutons, joystick...
gb_graphics gfx;         // l'objet "dessin"

extern "C" void app_main(void)   // le point d'entrée du programme
{
    gb.init();           // démarre l'écran, les entrées, les périphériques

    // (rien encore : on affichera quelque chose au chapitre 4)
    while (true) {
        gb.delay_ms(100);
    }
}
```

- `gb_core gb;` et `gb_graphics gfx;` sont des **objets** de la lib, déclarés une fois,
  en global, pour être accessibles partout.
- `gb.init()` s'appelle **une seule fois**, au tout début.
- La boucle `while (true)` empêche `app_main` de se terminer (sinon la console
  redémarrerait). Elle deviendra notre **boucle de jeu** au chapitre 5.

**À tester :** `idf.py build flash monitor`. La console démarre et ne plante pas. On ne
voit rien à l'écran pour l'instant : normal, on n'a rien dessiné.

---

## À retenir

- **CMake** décrit le projet ; ESP-IDF le découpe en **composants**, chacun avec son
  `CMakeLists.txt`.
- Dépendre de la lib = `REQUIRES gamebuino` (pas `INCLUDE_DIRS`).
- Le programme démarre dans `extern "C" void app_main(void)`.
- On garde `components/gamebuino` **intact**.

---

[« Précédent](Chapitre_02.md) | [Accueil](index.md) | [Suivant » : Premier affichage](Chapitre_04.md)
