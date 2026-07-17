# Chapitre 05 — Boucle de jeu  
[Accueil](index.md) | [Chapitre précédent](Chapitre_04.md) | [Chapitre suivant](Chapitre_06.md)

---

## 🏷️ Badges
![status](https://img.shields.io/badge/AKA-Tutoriel-blue)
![chapter](https://img.shields.io/badge/Chapitre-05-lightgrey)
![level](https://img.shields.io/badge/Niveau-Débutant-green)

---

## 📸 Illustration
![loop](img/chap05_loop.png)

---

# 📚 Table des matières
- [Objectif](#objectif)
- [Structure de la boucle](#structure-de-la-boucle)
- [Rendu](#rendu)
- [Limiteur de cadence](#limiteur-de-cadence)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🎯 Objectif
Créer une boucle de jeu stable et cadencée.

---

# 🔁 Structure de la boucle
```cpp
while (true) {
    read_input(k);
    update();
    draw();
}
---

# 🎨 Rendu
Le rendu se fait en fin de boucle via lcd_refresh().


# ⏱️ Limiteur de cadence
```cpp
vTaskDelay(pdMS_TO_TICKS(FRAME_MS - work));
```

# 🧠 À retenir
La boucle est le cœur du jeu.

Elle doit être stable et régulière.

Le timing est essentiel.

# 🚀 Pour aller plus loin
Ajouter un compteur FPS

Tester différentes cadences

# 🧭 Navigation
[Chapitre précédent](Chapitre_04.md) | [Chapitre suivant](Chapitre_06.md)
