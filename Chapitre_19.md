# Chapitre 19 — Menu Pause + Options

[« Précédent](Chapitre_18.md) | [Accueil](index.md) | [Suivant »](Chapitre_20.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Créer un **menu Pause** et un **menu Options** (volume, langue).

---

## 📸 Illustration
![menu](img/chap19_menu.png)

---

# 📚 Table des matières
- [Menu Pause](#menu-pause)
- [Menu Options](#menu-options)
- [Volume](#volume)
- [Langue](#langue)
- [Navigation](#navigation)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)

---

# ⏸️ Menu Pause

```cpp
int pause_index = 0;
const char* items[] = {"Reprendre","Options","Quitter"};
Navigation avec flèches.

⚙️ Menu Options
cpp
int options_index = 0;
const char* items[] = {"Volume","Langue","Retour"};
🔊 Volume
cpp
if (options_index==0 && k.left)  save.volume -= 0.05f;
if (options_index==0 && k.right) save.volume += 0.05f;
player.set_master_volume(save.volume);
save_save(save);
```

# 🌍 Langue
```cpp
if (options_index==1 && k.a_press)
    current_lang = (current_lang==Lang::FR)?Lang::EN:Lang::FR;
```

# 🧠 À retenir
Pause = Reprendre / Options / Quitter.

Options = Volume / Langue / Retour.

Volume sauvegardé.

Langue sauvegardée.

# 🚀 Pour aller plus loin
Ajouter un menu « Contrôles ».

Ajouter un menu « Graphismes ».

# 🧭 Navigation
[« Précédent](Chapitre_18.md) | [Suivant »](Chapitre_20.md)
