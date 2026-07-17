# Chapitre 09 — Balle  
[Accueil](index.md) | [Chapitre précédent](Chapitre_08.md) | [Chapitre suivant](Chapitre_10.md)

---

## 🏷️ Badges
![status](https://img.shields.io/badge/AKA-Tutoriel-blue)
![chapter](https://img.shields.io/badge/Chapitre-09-lightgrey)
![level](https://img.shields.io/badge/Niveau-Intermédiaire-yellow)

---

## 📸 Illustration
![ball](img/chap09_ball.png)

---

# 📚 Table des matières
- [Objectif](#objectif)
- [Structure de la balle](#structure-de-la-balle)
- [Mise à jour](#mise-à-jour)
- [Rebonds sur murs](#rebonds-sur-murs)
- [Dessin](#dessin)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🎯 Objectif
Créer une balle avec une physique simple mais stable.

---

# 🧱 Structure de la balle
```cpp
struct Ball : Rect {
    float vx, vy;
    bool active;
};
```

# 🔁 Mise à jour
```cpp
b.x += b.vx;
b.y += b.vy;
```

# 🧱 Rebonds sur murs
```cpp
if (b.x < 0 || b.x > SCREEN_W - BALL_SIZE) b.vx = -b.vx;
if (b.y < 0) b.vy = -b.vy;
```

# 🎨 Dessin
```cpp
gb_graphics.draw_rect(b.x, b.y, b.w, b.h, color_white);
```

# 🧠 À retenir
La balle doit être stable.

Les rebonds doivent être cohérents.

La vitesse doit être ajustée selon la cadence.

# 🚀 Pour aller plus loin
Ajouter un sprite rond

Ajouter une traînée lumineuse

# 🧭 Navigation
[Chapitre précédent](Chapitre_08.md) | [Chapitre suivant](Chapitre_10.md)
