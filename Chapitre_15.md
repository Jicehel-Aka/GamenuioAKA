# Chapitre 15 — Niveaux + briques incassables

[« Précédent](Chapitre_14.md) | [Accueil](index.md) | [Suivant »](Chapitre_16.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Créer des **niveaux structurés**, avec des **patterns** et des **briques incassables**.

---

## 📸 Illustration
![levels](img/chap15_levels.png)

---

# 📚 Table des matières
- [Structure d’un niveau](#structure-dun-niveau)
- [Pattern](#pattern)
- [Briques incassables](#briques-incassables)
- [Génération du niveau](#génération-du-niveau)
- [Collision](#collision)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🧱 Structure d’un niveau

```cpp
struct Level {
    int rows;
    int hp;
    std::vector<int> pattern;
};
🧩 Pattern
Code
0 = brique normale
1 = brique incassable
2 = vide
```

# 🧱 Briques incassables
```cpp
struct Brick {
    Rect r;
    int hp;
    bool alive;
    bool indestructible;
    uint16_t color;
};
```

# 🏗️ Génération du niveau
```cpp
Brick b;
b.indestructible = (code == 1);
b.hp = b.indestructible ? 999 : lvl.hp;
```

# 💥 Collision
```cpp
if (!b.indestructible)
    if (--b.hp <= 0) b.alive = false;
```

# 🧠 À retenir
Un niveau = un pattern.

Les incassables ont un HP très élevé.

Le code reste simple et propre.

# 🚀 Pour aller plus loin
Ajouter des briques explosives.

Ajouter des briques qui lâchent des bonus spécifiques.

Ajouter des briques animées.

# 🧭 Navigation
[« Précédent](Chapitre_14.md) | [Suivant »](Chapitre_16.md)
