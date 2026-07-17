# Chapitre 13 — Audio

[« Précédent](Chapitre_12.md) | [Accueil](index.md) | [Suivant »](Chapitre_14.md)

![Niveau](https://img.shields.io/badge/Niveau-Intermédiaire-yellow)

---

## Objectif

Ajouter un **système audio propre**, basé sur une **tâche dédiée** (FreeRTOS) et un
**mixage simple**. On joue plusieurs sons en même temps.

---

## 📸 Illustration
![audio](img/chap13_audio.png)

---

# 📚 Table des matières
- [Pourquoi une tâche audio](#pourquoi-une-tâche-audio)
- [Structure d’un son](#structure-dun-son)
- [Mixage](#mixage)
- [Tâche audio](#tâche-audio)
- [Jouer un son](#jouer-un-son)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🔊 Pourquoi une tâche audio ?

Si on joue les sons dans la boucle de jeu :

- micro‑lags  
- sons coupés  
- irrégularité  

Une tâche dédiée :

- lit les buffers audio  
- mixe les pistes  
- envoie au DAC  
- tourne indépendamment du jeu

---

# 🎵 Structure d’un son

```cpp
struct Sound {
    const int16_t* data;
    int length;
    int pos;
    bool playing;
};
```

# 🎚️ Mixage
``` cpp
int16_t mix() {
    int32_t acc = 0;
    for (auto& s : sounds)
        if (s.playing)
            acc += s.data[s.pos++];
    return acc >> 2; // limiter
}
```
# 🧵 Tâche audio
```cpp
void audio_task(void*) {
    while (true) {
        int16_t sample = mix();
        gb.audio.write(sample);
    }
}
```

# ▶️ Jouer un son
```cpp
void play(Sound& s) {
    s.pos = 0;
    s.playing = true;
}
```

# 🧠 À retenir

Une tâche dédiée = audio stable.

Le mixage additionne les pistes.

Le DAC reçoit un flux constant.

# 🚀 Pour aller plus loin
Ajouter des variations de pitch.

Ajouter un son de boss.

Ajouter une musique.

# 🧭 Navigation
[« Précédent](Chapitre_12.md) | [Accueil](index.md) | [Suivant »](Chapitre_14.md)
