# Chapitre 17 — Sauvegarde sur carte SD

[« Précédent](Chapitre_16.md) | [Accueil](index.md) | [Suivant »](Chapitre_18.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Sauvegarder le **score**, le **niveau**, le **volume**, la **langue** dans un fichier
texte simple sur la carte SD.

---

## 📸 Illustration
![save](img/chap17_save.png)

---

# 📚 Table des matières
- [Format du fichier](#format-du-fichier)
- [Structure SaveData](#structure-savedata)
- [Lecture](#lecture)
- [Écriture](#écriture)
- [Intégration](#intégration)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 📁 Format du fichier

score=12345
level=4
volume=0.80
lang=FR

Code

---

# 🧱 Structure SaveData

```cpp
struct SaveData {
    int best_score = 0;
    int level = 1;
    float volume = 0.8f;
    std::string lang = "FR";
};
```

# 📥 Lecture
```cpp
bool load_save(SaveData& s) {
    FILE* f = fopen("/game/saves/save.txt", "r");
    if (!f) return false;

    char key[32], val[32];
    while (fscanf(f, "%31[^=]=%31s\n", key, val) == 2) {
        if (!strcmp(key,"score")) s.best_score = atoi(val);
        else if (!strcmp(key,"level")) s.level = atoi(val);
        else if (!strcmp(key,"volume")) s.volume = atof(val);
        else if (!strcmp(key,"lang")) s.lang = val;
    }
    fclose(f);
    return true;
}
```

# 📤 Écriture
```cpp
void save_save(const SaveData& s) {
    FILE* f = fopen("/game/saves/save.txt", "w");
    fprintf(f,"score=%d\n",s.best_score);
    fprintf(f,"level=%d\n",s.level);
    fprintf(f,"volume=%.2f\n",s.volume);
    fprintf(f,"lang=%s\n",s.lang.c_str());
    fclose(f);
}
```

# 🔁 Intégration
```cpp
load_save(save);
...
save_save(save);
```
# 🧠 À retenir
Format texte = simple, robuste.

Lecture avec fscanf.

Sauvegarde automatique en fin de partie.

# 🚀 Pour aller plus loin
Sauvegarder les statistiques.

Sauvegarder les options graphiques.

Sauvegarder les bonus débloqués.

# 🧭 Navigation
[« Précédent](Chapitre_16.md) | [Accueil](index.md) | [Suivant »](Chapitre_18.md)
