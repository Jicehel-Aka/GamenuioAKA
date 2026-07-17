# Créer un jeu sur Gamebuino AKA — tutoriel pas à pas

Un tutoriel **pour débutant**, qui construit un jeu **de zéro** sur la Gamebuino AKA
(ESP32-S3, écran 320×240, C++/ESP-IDF), une brique à la fois. Chaque chapitre est court
et se termine par un programme **testable sur la console**.

On avance **par couches** : on comprend d'abord le principe « à la main » (le pixel, la
collision, le rebond…), **puis** on utilise les fonctions toutes prêtes de la
bibliothèque. Rien de « magique » : on sait toujours ce qui se passe en dessous.

> **Bibliothèque utilisée.** On s'appuie sur l'API **de haut niveau** de la lib AKA
> (`gb_core`, `gb_graphics`, audio…), jamais sur les pilotes bas niveau `gb_ll_…`. Le
> dossier `components/gamebuino` est une **brique externe** : on le prend **à jour** et
> on ne le modifie pas. Dépôt : <https://github.com/jmp42/Gamebuino_AKA_lib>

---

## Table des chapitres

**Les fondations (débutant)**

1. [Introduction](Chapitre_01.md) — le projet, la console, les deux couches d'outils
2. [Installer l'environnement](Chapitre_02.md) — ESP-IDF, `idf.py`, build/flash/monitor
3. [Structure du projet](Chapitre_03.md) — les `CMakeLists.txt` expliqués, le point d'entrée
4. [Premier affichage](Chapitre_04.md) — **du pixel au sprite** : framebuffer, couleurs, lignes, rectangles, transparence
5. [La boucle de jeu](Chapitre_05.md) — lire → mettre à jour → dessiner

**Le moteur (débutant → intermédiaire)**

6. [Cadence et timing](Chapitre_06.md) — une vitesse de jeu stable
7. [Lire les entrées](Chapitre_07.md) — joystick, boutons, **maintien vs déclenchement**
8. [La raquette](Chapitre_08.md) — la notion de `struct`, la primitive `Rect`, le *clamping*
9. [La balle](Chapitre_09.md) — position, vitesse, rebonds sur les murs
10. [Collision et rebond Arkanoid](Chapitre_10.md) — collision **générique**, rebond **progressif** (zones → % → sinus)

**La suite (à venir)**

11. Les briques — 12. *(option)* Machine à états — 13. Audio — 14. Bonus —
15. Niveaux + incassables — 16. Génération procédurale — 17. Sauvegarde SD —
18. Texte & multilingue — 19. Menu Pause + Options — 20. Optimisations —
21. Assemblage final — Bonus : particules, sprites animés, etc.

> ℹ️ La **machine à états** (chapitre 12) n'est **pas un prérequis** : c'est **une**
> manière d'organiser les écrans (titre, jeu, game over) parmi d'autres. On la présente
> comme une option pratique, pas comme un passage obligé.

---

## Conventions du code

- Deux objets globaux : `gb_core gb;` (console) et `gb_graphics gfx;` (dessin).
- Démarrage dans `extern "C" void app_main(void)`, `gb.init()` une seule fois.
- Couleurs : prédéfinies (`color_white`…) ou `gfx.makeColor(r, g, b)`.
- Textes **sans accent** (la police est ASCII).

---

## Licence

Voir [LICENSE](LICENSE).
