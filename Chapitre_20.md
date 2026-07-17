# Chapitre 20 — Optimisations

[« Précédent](Chapitre_19.md) | [Accueil](index.md) | [Suivant »](Chapitre_21.md)

![Niveau](https://img.shields.io/badge/Niveau-Avancé-red)

---

## Objectif

Optimiser le jeu : clipping, DMA, batching, sprites, performance.

---

## 📸 Illustration
![optim](img/chap20_optim.png)

---

# 📚 Table des matières
- [Clipping](#clipping)
- [DMA](#dma)
- [Sprites pré‑convertis](#sprites-pré-convertis)
- [Batching](#batching)
- [Éviter les divisions](#éviter-les-divisions)
- [Éviter les allocations](#éviter-les-allocations)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# ✂️ Clipping

Automatique via `gb_graphics`.

---

# ⚡ DMA

`gfx.update()` utilise le DMA → transfert rapide.

---

# 🎨 Sprites pré‑convertis

Convertir en RGB565 **avant** compilation.

---

# 📦 Batching

Dessiner les objets par groupes :

```cpp
paddle_draw();
for (auto& b : balls) ball_draw(b);
for (auto& k : bricks) brick_draw(k);
➗ Éviter les divisions
cpp
int cx = x + (w >> 1);
```
# 🚫 Éviter les allocations
```cpp
bricks.reserve(64);
```
# 🧠 À retenir
Clipping automatique.

DMA rapide.

Sprites pré‑convertis.

Pas d’allocations en jeu.

#🚀 Pour aller plus loin
Ajouter un système de particules optimisé.

Ajouter un moteur de shaders CPU.

# 🧭 Navigation
[« Précédent](Chapitre_19.md) | [Suivant »](Chapitre_21.md)
