# Chapitre 18 — Texte, polices, multilingue

[« Précédent](Chapitre_17.md) | [Accueil](index.md) | [Suivant »](Chapitre_19.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Gérer le texte, les accents, et plusieurs langues.  
La police intégrée est **ASCII uniquement** → pas d’accents.

---

## 📸 Illustration
![text](img/chap18_text.png)

---

# 📚 Table des matières
- [ASCII uniquement](#ascii-uniquement)
- [Police personnalisée](#police-personnalisée)
- [Système multilingue](#système-multilingue)
- [Intégration](#intégration)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🔤 ASCII uniquement

Écrire :

- « Ecran »  
- « Reessayer »  
- « Terminee »  

---

# 🎨 Police personnalisée (optionnel)

Sprite atlas + découpe :

```cpp
void draw_char(int x,int y,char c) {
    int idx = c - 32;
    int cx = (idx%16)*CHAR_W;
    int cy = (idx/16)*CHAR_H;
    gfx.drawSpriteRegion(x,y,font,cx,cy,CHAR_W,CHAR_H);
}
```

# 🌍 Système multilingue
```cpp
enum class Lang { FR, EN };
Lang current_lang = Lang::FR;

std::map<std::string,std::string> fr = {
    {"title","Appuyez sur A pour jouer"},
    {"gameover","Partie terminee"}
};

std::map<std::string,std::string> en = {
    {"title","Press A to play"},
    {"gameover","Game Over"}
};

const char* T(const std::string& key) {
    return (current_lang==Lang::FR)?fr[key].c_str():en[key].c_str();
}
```

# 🔁 Intégration
```cpp
gfx.print_str(T("title"));
```

# 🧠 À retenir
ASCII = pas d’accents.

Police custom = sprite atlas.

Dictionnaire = multilingue propre.

# 🚀 Pour aller plus loin
Ajouter l’espagnol, l’allemand, l’italien.

Ajouter une police pixel‑art.

# 🧭 Navigation
[« Précédent](Chapitre_17.md) | [Accueil](index.md) | [Suivant »](Chapitre_19.md)
