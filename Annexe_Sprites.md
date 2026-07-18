# Annexe B — Déboguer sur la Gamebuino AKA

[Accueil](index.md) | Utile dès le [Chapitre 02](Chapitre_02.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Un jeu ne compile jamais du premier coup, et ne marche jamais parfaitement au premier
flash. Cette annexe rassemble les **réflexes** qui font gagner des heures : lire les
erreurs dans le bon ordre, utiliser le moniteur série, et reconnaître les pannes
classiques.

---

## Règle d'or : lis la PREMIÈRE erreur, pas la dernière

Le compilateur C++ peut cracher des **centaines** d'erreurs à partir d'**une seule** faute.
Exemple vécu : une ligne de texte français oubliée **hors** d'un commentaire dans un `.h`…

```cpp
// dans fruits.h, ligne 17 :
tile_r/tile_c/prev_tile_/pixel_offset/dir   // <-- du texte, mais SANS // devant !
```

…déclenche un déluge du genre :

```
error: 'pixel_offset' does not name a type
error: 'size_t' has not been declared
error: 'ptrdiff_t' does not name a type
... (des centaines de lignes venant de <vector>, <type_traits>, <new> ...)
```

Ces centaines de lignes ne sont **pas** des vrais problèmes : dès qu'une erreur de syntaxe
survient **avant** que les en-têtes standards finissent d'être lus, le compilateur se
**désynchronise** et tout ce qui suit devient du charabia à ses yeux.

**La méthode :** remonte tout en haut, corrige la **première** erreur (ici : remettre la
ligne 17 dans un commentaire), recompile. Le plus souvent, les centaines d'autres
disparaissent d'un coup. Ne perds jamais de temps sur les erreurs du milieu ou de la fin
avant d'avoir traité la première.

---

## Ton meilleur outil : `printf` + le moniteur série

`idf.py monitor` (ou `flash monitor`) ouvre la **console série** : tout ce que tu écris
avec `printf(...)` s'y affiche pendant que le jeu tourne. C'est LA façon de voir ce qui se
passe « à l'intérieur ».

```cpp
printf("balle x=%.1f y=%.1f  vies=%d\n", ball.x, ball.y, g.lives);
printf("combo run=%d menu=%d\n", (int)k.run, (int)k.menu);   // débogage du retour loader
```

- Affiche les valeurs qui te surprennent (position, état, résultat d'un test).
- Un `printf` bien placé répond à la question « est-ce que ce code est seulement exécuté ? »
- On quitte le moniteur avec `Ctrl+]`.

> Astuce « bissection » : si tu ne sais pas d'où vient un bug, mets un `printf("ici 1\n")`,
> `printf("ici 2\n")`… à des points clés. Le dernier « ici N » affiché te dit **jusqu'où**
> le programme arrive avant de planter.

---

## Le tableau des pannes classiques

| Symptôme | Cause probable | Correctif |
|---|---|---|
| `undefined reference to ...` (à l'édition de liens) | dépendance non liée | mettre `gamebuino` dans **`REQUIRES`** (pas `INCLUDE_DIRS`), ch. 3 ; et **chaque `.cpp`** dans `SRCS` |
| `'X' does not name a type` en cascade | texte/typo hors commentaire, ou `#include` manquant dans un `.h` | corriger la **première** erreur (voir plus haut) |
| Redémarrages en boucle / `Guru Meditation` / `LoadProhibited` | accès mémoire invalide : indice de tableau hors bornes, pointeur nul | vérifier les bornes (`framebuffer` : 0..319 / 0..239), ne pas déréférencer un `nullptr` |
| `Stack overflow in task ...` | trop de données **locales** (gros tableau dans une fonction) ou récursion profonde | déplacer les gros tableaux en **`static`** ou en mémoire dynamique (PSRAM) ; agrandir la pile de la tâche |
| **Écran noir** alors que tu dessines | tu as oublié `gfx.update()` | appeler `gfx.update()` **une fois** en fin d'image (ch. 4-5) |
| **Aucun son** ou son haché | `player.pool()` pas assez fréquent, volume à 0, ou > 4 voix | tâche audio toutes les ~2 ms, `set_master_volume(200)`, max **4** pistes (ch. 13) |
| **Pas de score / sauvegarde impossible** | double montage de la carte SD, ou nom de fichier non-8.3 | laisser `gb.init()` monter la SD (ne pas remonter), noms **8.3** (ch. 17) |
| La raquette **dérive** toute seule | pas de zone morte sur le joystick | ignorer `|jx| < 200` (ch. 7) |
| La balle **traverse** ou **colle** à une brique | rebond sans repositionnement, ou collision multiple | replacer la balle au bord après rebond, `break` après la 1re brique (ch. 9-11) |
| Le combo **RUN+MENU** ne réagit pas | mauvaise touche testée, ou test hors de tous les états | lire `KEY_RUN`/`KEY_MENU`, appeler la vérif **chaque image, dans tous les états** (ch. 21) |

---

## Compiler et repartir propre

Quand des erreurs deviennent incompréhensibles (surtout après avoir changé de cible ou la
config), un **nettoyage** résout souvent le problème :

```bash
idf.py fullclean     # efface les fichiers de build
idf.py build         # reconstruit tout à neuf
```

Et si un flash se passe mal, vérifie le **port** (`-p COMx`) et que la console est bien en
mode réception.

---

## L'état d'esprit

- **Un bug à la fois.** Ne change pas dix choses en même temps : tu ne saurais pas laquelle
  a corrigé (ou cassé) quoi.
- **Reproduis d'abord.** Trouve les étapes exactes qui déclenchent le bug avant de le
  traquer.
- **Reviens à une version qui marche.** Garde des sauvegardes (ou un dépôt Git) : pouvoir
  comparer « avant/après » vaut de l'or.

---

## À retenir

- **Toujours** corriger la **première** erreur du compilateur ; les suivantes en découlent
  souvent.
- `printf` + **moniteur série** = ta lampe torche ; la « bissection » localise un plantage.
- La plupart des pannes sont dans le **tableau** ci-dessus : `undefined reference`, écran
  noir, son muet, SD, dérive joystick, combo loader.

---

[Accueil](index.md) | [Chapitre 02 — Installer l'environnement](Chapitre_02.md)
