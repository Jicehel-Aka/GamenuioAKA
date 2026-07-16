# Chapitre 02 — Installation du SDK  
[Accueil](index.md) | [Chapitre précédent](Chapitre_01.md) | [Chapitre suivant](Chapitre_03.md)

---

## 🏷️ Badges
![status](https://img.shields.io/badge/AKA-Tutoriel-blue)
![chapter](https://img.shields.io/badge/Chapitre-02-lightgrey)
![level](https://img.shields.io/badge/Niveau-Débutant-green)

---

## 📸 Illustration
![sdk](img/chap02_sdk.png)

---

# 📚 Table des matières
- [Objectif](#objectif)
- [Téléchargement du SDK](#téléchargement-du-sdk)
- [Installation de la toolchain](#installation-de-la-toolchain)
- [Compilation de test](#compilation-de-test)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🎯 Objectif
Installer l’environnement de développement complet pour programmer sur la Gamebuino AKA.

---

# 📥 Téléchargement du SDK
Téléchargez le SDK officiel Gamebuino AKA depuis le site ou le dépôt GitHub.

---

# 🛠️ Installation de la toolchain ESP32‑S3
Installez :

- CMake  
- Ninja  
- Python 3  
- ESP-IDF (version compatible AKA)  

---

# 🧪 Compilation de test
```bash
mkdir build
cd build
cmake ..
ninja
