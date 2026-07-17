# Chapitre Bonus — Particules, Boss, Shaders, Sprites animés

[« Précédent](Chapitre_21.md) | [Accueil](index.md)

![Niveau](https://img.shields.io/badge/Niveau-Expert-purple)

---

## Objectif

Ajouter des effets avancés : particules, boss, shaders CPU, sprites animés.

---

## 📸 Illustration
![bonus](img/chap_bonus.png)

---

# 📚 Table des matières
- [Particules](#particules)
- [Boss](#boss)
- [Shaders CPU](#shaders-cpu)
- [Sprites animés](#sprites-animés)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# ✨ Particules

```cpp
struct Particle {
    float x,y,vx,vy;
    int life;
    uint16_t col;
};
```

# 👹 Boss
``` cpp
struct Boss : Rect {
    int hp;
    float vx;
};
```

# 🎨 Shaders CPU
Effets visuels simples :

flash blanc

teinte globale

vignette

# 🖼️ Sprites animés
```cpp
struct AnimSprite {
    const Sprite* frames[4];
    int frame, frame_count, timer, speed;
};
```

# 🧠 À retenir
Particules = vie visuelle.

Boss = gameplay avancé.

Shaders CPU = effets légers.

Sprites animés = polish.

# 🚀 Pour aller plus loin
Ajouter un mode Boss Rush.

Ajouter un système de particules GPU‑like.

Ajouter des animations de briques.

# 🧭 Navigation
[« Précédent](Chapitre_21.md) | [Accueil](index.md)
