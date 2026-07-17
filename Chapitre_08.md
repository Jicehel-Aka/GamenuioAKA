# Chapitre 08 — Raquette  
[Accueil](index.md) | [Chapitre précédent](Chapitre_07.md) | [Chapitre suivant](Chapitre_09.md)

---

## 🏷️ Badges
![status](https://img.shields.io/badge/AKA-Tutoriel-blue)
![chapter](https://img.shields.io/badge/Chapitre-08-lightgrey)
![level](https://img.shields.io/badge/Niveau-Intermédiaire-yellow)

---

## 📸 Illustration
![paddle](img/chap08_paddle.png)

---

# 📚 Table des matières
- [Objectif](#objectif)
- [Structure de la raquette](#structure-de-la-raquette)
- [Mise à jour](#mise-à-jour)
- [Clamping](#clamping)
- [Dessin](#dessin)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🎯 Objectif
Créer une raquette fluide et réactive.

---

# 🧱 Structure de la raquette
```cpp
struct Paddle : Rect {
    float speed;
};
```

# 🔁 Mise à jour
```cpp
if (keys.left)  p.x -= p.speed;
if (keys.right) p.x += p.speed;
``


# 🧲 Clamping
```cpp
p.x = std::clamp(p.x, 0.0f, SCREEN_W - PADDLE_W);
```

# 🎨 Dessin
```cpp
gb_graphics.draw_rect(p.x, p.y, p.w, p.h, color_white);
``


# 🧠 À retenir
La raquette doit être fluide.

Le clamping évite les sorties d’écran.

La vitesse doit être ajustée selon la cadence.

# 🚀 Pour aller plus loin
Ajouter une animation de glow

Ajouter un sprite animé

# 🧭 Navigation
[Chapitre précédent](Chapitre_07.md) | [Chapitre suivant](Chapitre_09.md)
