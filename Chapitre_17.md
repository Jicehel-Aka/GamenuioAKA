# Chapitre 17 — Sauvegarder sur la carte SD

[« Précédent](Chapitre_16.md) | [Accueil](index.md) | [Suivant »](Chapitre_18.md)

![Niveau](https://img.shields.io/badge/Niveau-Interm%C3%A9diaire-yellow)

---

## Objectif

Conserver le **meilleur score** et les **réglages** (volume, langue) d'une partie à
l'autre, dans un fichier sur la carte SD.

---

## La carte est déjà montée

Bonne nouvelle : `gb.init()` (chapitre 3) **monte** déjà la carte SD. Elle est accessible
sous le chemin **`/sdcard`**. Tu écris donc dans des fichiers comme
`/sdcard/AKABRICK/SAVE.TXT`, avec les fonctions de fichiers standard du C.

> ⚠️ **Piège vécu** : ne **re-monte pas** la carte toi-même. Un second montage sous une
> carte déjà montée **casse l'accès** aux fichiers (symptôme : « aucun score », sauvegarde
> impossible). On laisse `gb.init()` s'en charger, un point c'est tout.

### Noms de fichiers : la règle du 8.3

Le système de fichiers de la carte est de type FAT. Pour éviter tout souci, garde des
noms **8.3** : au plus **8 caractères** + un point + **3** d'extension, en majuscules.
`SAVE.TXT`, `SCORES.DAT`, un dossier `AKABRICK`… sont sûrs.

> ℹ️ La console n'a **pas d'horloge réglée** : les fichiers n'auront pas de date de
> modification correcte. C'est **normal**, pas un bug.

---

## Lire et écrire un fichier, pour un débutant

En C, un fichier se manipule via un **`FILE*`** obtenu avec `fopen`. On précise le mode :
`"w"` (écrire, écrase), `"r"` (lire), `"a"` (ajouter). On **doit** refermer avec `fclose`.

```cpp
FILE* f = fopen("/sdcard/AKABRICK/SAVE.TXT", "w");   // ouvrir en écriture
if (f) {                                             // toujours vérifier !
    fprintf(f, "Bonjour\n");                         // écrire comme un printf
    fclose(f);                                       // refermer (important)
}
```

- `fprintf(f, ...)` écrit **dans le fichier** (au lieu de l'écran).
- `fscanf(f, ...)` lit **depuis le fichier**.
- Si `fopen` échoue, il renvoie `nullptr` : on teste **toujours** avant d'écrire/lire.

---

## Un format texte, simple et lisible

Pour débuter, un fichier **texte** « clé = valeur » est parfait : facile à écrire, à
relire, et même à ouvrir sur un PC pour vérifier.

```
score=12345
volume=200
lang=FR
```

### La structure des données à sauver

```cpp
struct SaveData {
    int  best_score = 0;
    int  volume     = 200;      // 0..255 (chapitre 13)
    char lang[3]    = "FR";     // "FR" ou "EN"
};
```

### Écrire

```cpp
void save_write(const SaveData& s) {
    ensure_dir("/sdcard/AKABRICK");                  // crée le dossier si besoin (voir plus bas)
    FILE* f = fopen("/sdcard/AKABRICK/SAVE.TXT", "w");
    if (!f) { printf("SD: echec ecriture\n"); return; }
    fprintf(f, "score=%d\n",  s.best_score);
    fprintf(f, "volume=%d\n", s.volume);
    fprintf(f, "lang=%s\n",   s.lang);
    fclose(f);
}
```

### Lire

```cpp
bool save_read(SaveData& s) {
    FILE* f = fopen("/sdcard/AKABRICK/SAVE.TXT", "r");
    if (!f) return false;                            // pas de fichier = première partie
    char cle[16], val[16];
    // format "%15[^=]=%15s" : lire jusqu'a 15 caracteres qui ne sont PAS '=' (la clé),
    // puis le '=', puis jusqu'a 15 caracteres sans espace (la valeur).
    // fscanf renvoie le nombre de champs lus : on continue tant qu'on en lit bien 2.
    while (fscanf(f, "%15[^=]=%15s\n", cle, val) == 2) {   // lit "cle=val" ligne à ligne
        if      (!strcmp(cle, "score"))  s.best_score = atoi(val);
        else if (!strcmp(cle, "volume")) s.volume     = atoi(val);
        else if (!strcmp(cle, "lang"))   { s.lang[0]=val[0]; s.lang[1]=val[1]; s.lang[2]=0; }
    }
    fclose(f);
    return true;
}
```

`atoi(val)` convertit le texte `"12345"` en nombre `12345` ; `strcmp` compare deux
chaînes (renvoie 0 si égales).

---

## Créer le dossier si besoin

Avant d'écrire, le dossier `AKABRICK` doit exister. On le crée avec `mkdir` (sans erreur
s'il est déjà là) :

```cpp
#include <sys/stat.h>
void ensure_dir(const char* path) {
    mkdir(path, 0777);        // crée le dossier ; ne fait rien s'il existe déjà
}
```

---

## Quand sauver / charger ?

```cpp
extern "C" void app_main(void) {
    gb.init();
    SaveData save;
    save_read(save);          // au démarrage : on récupère l'existant

    // ... jeu ...
    // à la fin d'une partie, si on bat le record :
    if (g.score > save.best_score) { save.best_score = g.score; save_write(save); }
    // et à chaque changement de réglage (volume, langue) : save_write(save);
}
```

> 💡 Charge les réglages **après** le montage SD (donc après `gb.init()`) et **avant**
> d'appliquer le volume (`player.set_master_volume(save.volume)`), pour partir sur les
> bonnes valeurs.

**À tester :** fais un score, éteins/rallume la console : le meilleur score et tes
réglages sont toujours là. Ouvre `SAVE.TXT` sur un PC pour voir le contenu en clair.

---

## À retenir

- La SD est montée par `gb.init()` sous **`/sdcard`** : **ne la remonte pas**.
- Noms de fichiers **8.3** ; pas de date de fichier fiable (pas d'horloge).
- `fopen`/`fprintf`/`fscanf`/`fclose`, en **vérifiant** toujours `fopen` ; un format
  **texte** « clé=valeur » est idéal pour débuter.

---

[« Précédent](Chapitre_16.md) | [Accueil](index.md) | [Suivant » : Texte & multilingue](Chapitre_18.md)
