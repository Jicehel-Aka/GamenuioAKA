# Chapitre 07 — Lecture des entrées  
[Accueil](index.md) | [Chapitre précédent](Chapitre_06.md) | [Chapitre suivant](Chapitre_08.md)

---

## 🏷️ Badges
![status](https://img.shields.io/badge/AKA-Tutoriel-blue)
![chapter](https://img.shields.io/badge/Chapitre-07-lightgrey)
![level](https://img.shields.io/badge/Niveau-Intermédiaire-yellow)

---

## 📸 Illustration
![inputs](img/chap07_inputs.png)

---

# 📚 Table des matières
- [Objectif](#objectif)
- [Structure des entrées](#structure-des-entrées)
- [Lecture du joystick](#lecture-du-joystick)
- [Lecture des boutons](#lecture-des-boutons)
- [Gestion des “pressed”](#gestion-des-pressed)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🎯 Objectif
Lire proprement les entrées du joueur pour contrôler la raquette et les menus.

---

# 🧩 Structure des entrées
```cpp
struct Keys {
    uint16_t pressed;
    bool left, right;
};
```

# 🎮 Lecture du joystick
```cpp
keys.left  = (joy_x < -0.3f);
keys.right = (joy_x >  0.3f);
```

# 🔘 Lecture des boutons
``` cpp
keys.pressed = expander_get_buttons_pressed();
```

# 🖱️ Gestion des “pressed”
Les actions doivent se déclencher sur front, pas en continu.

# 🧠 À retenir
Le joystick contrôle la raquette.

Les boutons contrôlent les menus.

Les “pressed” évitent les répétitions involontaires.

# 🚀 Pour aller plus loin
Ajouter un système de double‑tap

Ajouter un système de long‑press

# 🧭 Navigation
[Chapitre précédent](Chapitre_06.md) | [Chapitre suivant](Chapitre_08.md)
