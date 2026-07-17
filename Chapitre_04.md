# Chapitre 04 — Premier affichage  
[Accueil](index.md) | [Chapitre précédent](Chapitre_03.md) | [Chapitre suivant](Chapitre_05.md)

---

## 🏷️ Badges
![status](https://img.shields.io/badge/AKA-Tutoriel-blue)
![chapter](https://img.shields.io/badge/Chapitre-04-lightgrey)
![level](https://img.shields.io/badge/Niveau-Débutant-green)

---

## 📸 Illustration
![display](img/chap04_display.png)

---

# 📚 Table des matières
- [Objectif](#objectif)
- [Initialisation de l’écran](#initialisation-de-lécran)
- [Premier dessin](#premier-dessin)
- [Rafraîchissement](#rafraîchissement)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🎯 Objectif
Afficher un premier élément à l’écran pour valider le pipeline graphique.

---

# 🖥️ Initialisation de l’écran
```cpp
lcd_clear(color_black);
lcd_refresh();
```

# 🎨 Premier dessin
```cpp
gb_graphics.draw_rect(10, 10, 50, 50, color_white);
lcd_refresh();
```

# 🔁 Rafraîchissement
Le DMA envoie le framebuffer à l’écran automatiquement.

# 🧠 À retenir
Le rendu est simple et rapide.

Le DMA optimise le transfert.

Vous pouvez dessiner rectangles, textes, sprites.

# 🚀 Pour aller plus loin
Tester plusieurs couleurs
Dessiner du texte

# 🧭 Navigation
[Chapitre précédent](Chapitre_03.md) | [Chapitre suivant](Chapitre_05.md)
