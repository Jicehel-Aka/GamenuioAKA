# Chapitre 06 — Cadence et timing  
[Accueil](index.md) | [Chapitre précédent](Chapitre_05.md) | [Chapitre suivant](Chapitre_07.md)

---

## 🏷️ Badges
![status](https://img.shields.io/badge/AKA-Tutoriel-blue)
![chapter](https://img.shields.io/badge/Chapitre-06-lightgrey)
![level](https://img.shields.io/badge/Niveau-Intermédiaire-yellow)

---

## 📸 Illustration
![timing](img/chap06_timing.png)

---

# 📚 Table des matières
- [Objectif](#objectif)
- [Pourquoi une cadence fixe](#pourquoi-une-cadence-fixe)
- [Calcul du temps de frame](#calcul-du-temps-de-frame)
- [Limiteur de cadence](#limiteur-de-cadence)
- [Cadence recommandée](#cadence-recommandée)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🎯 Objectif
Garantir une cadence stable pour éviter les variations de vitesse et les micro‑lags.

---

# 🕒 Pourquoi une cadence fixe ?
Un jeu fluide doit :

- mettre à jour la logique à intervalle constant  
- dessiner à intervalle constant  
- éviter les frames trop rapides ou trop lentes  

---

# 🧮 Calcul du temps de frame
```cpp
uint32_t frame_start = esp_timer_get_time() / 1000;
```

# ⏱️ Limiteur de cadence
```cpp
uint32_t work = (esp_timer_get_time() / 1000) - frame_start;
if (work < FRAME_MS)
    vTaskDelay(pdMS_TO_TICKS(FRAME_MS - work));
```

# 🎚️ Cadence recommandée
30 FPS : idéal pour AKA

60 FPS : possible mais inutile pour un casse‑briques

# 🧠 À retenir
Une cadence fixe = gameplay stable.

Le timing est essentiel pour la physique.

Le limiter évite les variations de vitesse.

# 🚀 Pour aller plus loin
Ajouter un compteur FPS

Tester une cadence adaptative

# 🧭 Navigation
[Chapitre précédent](Chapitre_05.md) | [Chapitre suivant](Chapitre_07.md)
