# Créer un jeu sur Gamebuino AKA — tutoriel pas à pas

Un tutoriel **pour débutant** qui construit un casse-briques **de zéro** sur la Gamebuino
AKA (ESP32-S3, écran 320×240, C++/ESP-IDF), une brique à la fois. Chaque chapitre est
court et se termine par un programme **testable sur la console**.

On avance **par couches** : on comprend d'abord le principe « à la main » (le pixel, la
collision, le rebond, le blit…), **puis** on utilise les fonctions toutes prêtes de la
bibliothèque. Rien de « magique » : on sait toujours ce qui se passe en dessous.

> **Bibliothèque utilisée.** On s'appuie sur l'API **de haut niveau** de la lib AKA
> (`gb_core`, `gb_graphics`, `gb_audio_player`…), **jamais** sur les pilotes bas niveau
> `gb_ll_…`. Le dossier `components/gamebuino` est une **brique externe** : on le prend
> **à jour** et on ne le modifie pas. Dépôt : <https://github.com/jmp42/Gamebuino_AKA_lib>

---

## Table des chapitres

**Fondations (débutant)**

0. [Notions de C++ pour démarrer](Chapitre_00.md) — types, bits, `struct`/objets, références & pointeurs, mémoire
1. [Introduction](Chapitre_01.md) — le projet, la console, les deux couches d'outils
2. [Installer l'environnement](Chapitre_02.md) — ESP-IDF, `idf.py`, build/flash/monitor
3. [Structure du projet](Chapitre_03.md) — les `CMakeLists.txt` expliqués, le point d'entrée
4. [Premier affichage](Chapitre_04.md) — **du pixel au sprite** : framebuffer, couleurs, lignes, rectangles, transparence
5. [La boucle de jeu](Chapitre_05.md) — lire → mettre à jour → dessiner

**Le moteur (débutant → intermédiaire)**

6. [Cadence et timing](Chapitre_06.md) — une vitesse de jeu stable
7. [Lire les entrées](Chapitre_07.md) — joystick, boutons, **maintien vs déclenchement**
8. [La raquette](Chapitre_08.md) — la `struct`, la primitive `Rect`, le *clamping*
9. [La balle](Chapitre_09.md) — position, vitesse, rebonds sur les murs
10. [Collision et rebond Arkanoid](Chapitre_10.md) — collision **générique**, rebond **progressif** (zones → % → sinus)

**Le jeu complet (intermédiaire)**

11. [Les briques](Chapitre_11.md) — `std::vector`, grille, axe de rebond
12. [Organiser les écrans](Chapitre_12.md) — machine à états *(une option, pas un prérequis)*
13. [Le son](Chapitre_13.md) — `gb_audio_player`, tâche audio, voix + gain
14. [Les bonus qui tombent](Chapitre_14.md) — colle, laser, multi-ball
15. [Niveaux et incassables](Chapitre_15.md) — plans de niveau, briques indestructibles
16. [Génération procédurale](Chapitre_16.md) — motifs, *seed* reproductible
17. [Sauvegarde SD](Chapitre_17.md) — `/sdcard`, noms 8.3, fichiers texte
18. [Texte et multilingue](Chapitre_18.md) — ASCII, table `[langue][id]`
19. [Menu Pause et Options](Chapitre_19.md) — listes, volume, langue

**Finition (avancé)**

20. [Optimiser (quand c'est utile)](Chapitre_20.md) — mesurer d'abord, puis cibler
21. [Assemblage final et publication](Chapitre_21.md) — découpage en fichiers, retour loader, GitHub
- [Chapitre Bonus](Chapitre_bonus.md) — particules, sprites animés, effets plein écran, boss

**Annexes**

- [Annexe A — Créer et convertir un sprite](Annexe_Sprites.md) — dessiner → PNG → `.h` (script `tools/png2sprite.py`)
- [Annexe B — Déboguer sur la AKA](Annexe_Debug.md) — lire la 1re erreur, `printf` + moniteur, pannes classiques

---

## Conventions du code

- Deux objets globaux : `gb_core gb;` (console) et `gb_graphics gfx;` (dessin) ;
  `gb_audio_player player;` + `gb_audio_track_tone voices[4];` pour le son.
- Démarrage dans `extern "C" void app_main(void)`, `gb.init()` **une seule fois**.
- Couleurs : prédéfinies (`color_white`…) ou `gfx.makeColor(r, g, b)`.
- Entrées : `gb.pool()` une fois par image ; **maintien** (`state`) vs **front**
  (`pressed`).
- Volume : `player.set_master_volume(0..255)`. Fichiers SD sous `/sdcard`, noms **8.3**.
- Textes **sans accent** (police ASCII).

---

## Licence

Voir [LICENSE](LICENSE).
