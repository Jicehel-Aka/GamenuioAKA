# Chapitre 16 — Génération procédurale de niveaux

[« Précédent](Chapitre_15.md) | [Accueil](index.md) | [Suivant »](Chapitre_17.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Créer des **niveaux générés automatiquement**, à partir de **règles** et de **patterns**.
On ne veut pas du hasard pur, mais une génération **structurée**, jouable et variée.

---

## 📸 Illustration
![procedural](img/chap16_procedural.png)

---

# 📚 Table des matières
- [Procédural ≠ aléatoire](#procédural--aléatoire)
- [Patterns procéduraux](#patterns-procéduraux)
- [Générateur](#générateur)
- [Progression de difficulté](#progression-de-difficulté)
- [Intégration](#intégration)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🧠 Procédural ≠ aléatoire

Un bon niveau procédural :

- suit une **structure**  
- reste **lisible**  
- reste **jouable**  
- varie selon un **seed**  

On génère des patterns, pas du bruit.

---

# 🧩 Patterns procéduraux

Exemples :

### Damier
0 1 0 1 0 1 0 1 0
1 0 1 0 1 0 1 0 1

Code

### Mur central
0 0 0 1 1 1 0 0 0

Code

### Vague
0 0 1 1 1 0 0 0 0
1 1 1 1 1 1 0 0 0

Code

### Aléatoire contrôlé
0 ou 1 selon une probabilité

Code

---

# 🏗️ Générateur

```cpp
Level make_procedural_level(int seed) {
    srand(seed);
    Level lvl;
    lvl.rows = 6;
    lvl.hp = 1;
    lvl.pattern.resize(BRICK_ROWS * BRICK_COLS);

    for (int r = 0; r < lvl.rows; r++) {
        for (int c = 0; c < BRICK_COLS; c++) {
            int idx = r * BRICK_COLS + c;

            switch (seed % 4) {
                case 0: lvl.pattern[idx] = ((r+c)%2==0)?0:1; break;
                case 1: lvl.pattern[idx] = (c>=3&&c<=5)?1:0; break;
                case 2: lvl.pattern[idx] = (c < r+3)?1:0; break;
                case 3:
                    int roll = rand()%100;
                    lvl.pattern[idx] = (roll<10)?2:(roll<40)?1:0;
                    break;
            }
        }
    }
    return lvl;
}
```

# 📈 Progression de difficulté
```cpp
lvl.hp = 1 + level_number / 3;
```

# 🔁 Intégration
```cpp
Level lvl = make_procedural_level(rand());
bricks = make_level(lvl);
```
# 🧠 À retenir
Procédural = règles, pas hasard.

Patterns = variété contrôlée.

Seed = niveaux différents à chaque partie.

# 🚀 Pour aller plus loin
Ajouter des briques explosives.

Ajouter des patterns dynamiques.

Ajouter des niveaux boss.

# 🧭 Navigation
[« Précédent](Chapitre_15.md) | [Suivant »](Chapitre_17.md)
