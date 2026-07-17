# Chapitre 12 — Machine à états

[« Précédent](Chapitre_11.md) | [Accueil](index.md) | [Suivant »](Chapitre_13.md)

![Niveau](https://img.shields.io/badge/Niveau-Intermédiaire-yellow)

---

## Objectif

Organiser proprement les **écrans** du jeu : titre, jeu, pause, game over.  
Une machine à états évite les `if` partout et rend le code lisible.

---

## 📸 Illustration
![states](img/chap12_states.png)

---

# 📚 Table des matières
- [Pourquoi une machine à états](#pourquoi-une-machine-à-états)
- [Définition des états](#définition-des-états)
- [Structure Game](#structure-game)
- [Boucle principale](#boucle-principale)
- [Transitions](#transitions)
- [À retenir](#à-retenir)
- [Pour aller plus loin](#pour-aller-plus-loin)
- [Navigation](#navigation)

---

# 🎯 Pourquoi une machine à états ?

Sans machine à états :

- le code du titre mélange le code du jeu  
- les menus se mélangent avec la physique  
- les transitions deviennent confuses  

Avec une machine à états :

- chaque écran a sa fonction  
- la boucle principale est propre  
- les transitions sont explicites

---

# 🧩 Définition des états

```cpp
enum class State {
    Title,
    WaitingBall,
    Playing,
    Pause,
    GameOver
};
```

# 🧱 Structure Game
```cpp
struct Game {
    State state;
    Paddle paddle;
    Ball ball;
    std::vector<Brick> bricks;
    int score;
};
```

# 🔁 Boucle principale
```cpp
while (true) {
    read_input(k);

    switch (g.state) {
        case State::Title:       update_title(g, k); break;
        case State::WaitingBall: update_waiting(g, k); break;
        case State::Playing:     update_playing(g, k); break;
        case State::Pause:       update_pause(g, k); break;
        case State::GameOver:    update_gameover(g, k); break;
    }
}
```

#🔀 Transitions

Exemples :
```cpp
if (k.a_press) g.state = State::Playing;
if (ball_lost) g.state = State::GameOver;
if (k.menu)    g.state = State::Pause;
```

# 🧠 À retenir
Un état = un écran.

Une fonction par état = code propre.

Les transitions sont explicites et lisibles.

# 🚀 Pour aller plus loin
Ajouter un état « Options ».

Ajouter un état « Victoire ».

Ajouter un état « Boss ».


# 🧭 Navigation
[« Précédent](Chapitre_11.md) | [Suivant »](Chapitre_13.md)
