# Chapitre 14 — Bonus

[« Précédent](Chapitre_13.md) | [Accueil](index.md) | [Suivant »](Chapitre_15.md)

![Niveau](https://img.shields.io/badge/Niveau-Intermédiaire-yellow)

---

## Objectif

Ajouter les bonus classiques : **colle**, **laser**, **multi‑ball**.  
Ils tombent des briques et modifient le gameplay.

---

## 📸 Illustration
![bonus](img/chap14_bonus.png)

---

# 📚 Table des matières
- [Structure d’un bonus](#structure-dun-bonus)
- [Apparition](#apparition)
- [Chute](#chute)
- [Collision avec la raquette](#collision-avec-la-raquette)
- [Bonus colle](#bonus-colle)
- [Bonus laser](#bonus-laser)
- [Bonus multi‑ball](#bonus-multi-ball)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🧩 Structure d’un bonus

```cpp
enum class Power { Glue, Laser, Multi };

struct Falling {
    Rect r;
    Power type;
    bool active;
};
```

# 🎲 Apparition
```cpp
if (rand() % 6 == 0)
    falling.push_back({ {x, y, 12, 12}, Power::Glue, true });
```

# ⬇️ Chute
```cpp
f.r.y += 2;
if (f.r.y > SCREEN_H) f.active = false;
```

# 🧱 Collision avec la raquette
```cpp
if (overlap(f.r, paddle.r)) {
    apply_power(f.type);
    f.active = false;
}
```

# 🧲 Bonus colle
La balle reste collée à la raquette pendant un temps.

# 🔫 Bonus laser
La raquette tire des projectiles qui détruisent les briques.

#🔮 Bonus multi‑ball
On ajoute une seconde balle :

```cpp
Ball b2 = ball;
b2.vx = -b2.vx;
balls.push_back(b2);
```

# 🧠 À retenir
Les bonus enrichissent le gameplay.

La colle nécessite un timer.

Le laser ajoute des tirs.

Le multi‑ball duplique la balle.

# 🚀 Pour aller plus loin
Ajouter un bonus « slow motion ».

Ajouter un bonus « élargir la raquette ».

Ajouter un bonus « bombe ».

# 🧭 Navigation
[« Précédent](Chapitre_13.md) | [Suivant »](Chapitre_15.md)
