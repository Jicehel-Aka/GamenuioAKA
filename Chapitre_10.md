# Chapitre 10 — Collision Arkanoid  
[Accueil](index.md) | [Chapitre précédent](Chapitre_09.md) | [Chapitre suivant](Chapitre_11.md)

---

## 🏷️ Badges
![status](https://img.shields.io/badge/AKA-Tutoriel-blue)
![chapter](https://img.shields.io/badge/Chapitre-10-lightgrey)
![level](https://img.shields.io/badge/Niveau-Intermédiaire-yellow)

---

## 📸 Illustration
![arkanoid](img/chap10_arkanoid.png)

---

# 📚 Table des matières
- [Objectif](#objectif)
- [Collision balle/raquette](#collision-ballerquette)
- [Angle Arkanoid](#angle-arkanoid)
- [Correction de position](#correction-de-position)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🎯 Objectif
Implémenter la collision Arkanoid pour un rebond dynamique.

---

# 🎯 Collision balle/raquette
```cpp
if (collide(ball, paddle)) {
    float t = (ball.x - paddle.x) / paddle.w;
    ball.vx = (t - 0.5f) * 4.0f;
    ball.vy = -fabs(ball.vy);
}
```

# 🎚️ Angle Arkanoid
Le rebond dépend de la position d’impact.

# 🧱 Correction de position
```cpp
ball.y = paddle.y - BALL_SIZE;
```

# 🧠 À retenir
Arkanoid = rebond dynamique.

Le rebond dépend de la position d’impact.

La correction évite les collisions multiples.

# 🚀 Pour aller plus loin
Ajouter un effet sonore variable selon l’impact

Ajouter une animation de raquette

# 🧭 Navigation
[Chapitre précédent](Chapitre_09.md) | [Chapitre suivant](Chapitre_11.md)
