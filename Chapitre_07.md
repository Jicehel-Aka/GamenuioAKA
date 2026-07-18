# Chapitre 07 — Lire les entrées

[« Précédent](Chapitre_06.md) | [Accueil](index.md) | [Suivant »](Chapitre_08.md)

![Niveau](https://img.shields.io/badge/Niveau-D%C3%A9butant-green)

---

## Objectif

Lire proprement le **joystick** et les **boutons**, et surtout comprendre la différence
entre « la touche **est enfoncée** » et « la touche **vient d'être** enfoncée » — une
distinction qui évite énormément de bugs.

---

## Une seule lecture par image

Tout le matériel d'entrée se lit d'un coup avec **`gb.pool()`**. On l'appelle **une fois
par image**, au début de la boucle. Ensuite, on interroge l'état déjà récupéré autant de
fois qu'on veut, sans relire le matériel.

> ⚠️ Pourquoi une seule lecture ? Les boutons et le joystick partagent le même bus
> interne. Deux lectures concurrentes dans la même image peuvent se gêner. Règle simple :
> **un seul `gb.pool()` par image**, tout le monde lit le résultat.

---

## Les boutons

Après `gb.pool()`, l'objet `gb.buttons` répond à trois questions différentes :

```cpp
gb.pool();

bool a_maintenu    = gb.buttons.state()   & gb_buttons::KEY_A;   // A est-il MAINTENU ?
bool a_vient_appui = gb.buttons.pressed(gb_buttons::KEY_A);       // A vient-il d'être ENFONCÉ ?
```

> 💡 **Le `&` (« et » bit à bit) et le masque de touches.** `gb.buttons.state()` ne
> renvoie pas un seul bouton, mais **tous** d'un coup, empaquetés dans un nombre où
> **chaque bit** représente une touche (1 = enfoncée, 0 = relâchée). `gb_buttons::KEY_A`
> est justement le bit de la touche A. Le `&` (bit à bit) « éteint » tous les autres bits
> et ne garde que celui de A : le résultat est non nul (donc « vrai ») **uniquement** si A
> est enfoncée. C'est un **masque** : on isole une info au milieu des autres. (Voir le
> [Chapitre 0](Chapitre_00.md) pour les bits.)

En réalité la lib distingue trois notions. Le plus clair est d'utiliser les versions qui
prennent une touche et renvoient un `bool` :

| Question | Appel | Quand c'est vrai |
|---|---|---|
| La touche est-elle **maintenue** ? | `gb.buttons.state()` (masque) | à chaque image tant qu'on appuie |
| Vient-elle d'être **enfoncée** ? | `gb.buttons.pressed(KEY_x)` | **une seule image**, au moment de l'appui |
| Vient-elle d'être **relâchée** ? | `gb.buttons.released(KEY_x)` | **une seule image**, au moment du relâcher |

Les touches disponibles : `KEY_LEFT`, `KEY_RIGHT`, `KEY_UP`, `KEY_DOWN`, `KEY_A`,
`KEY_B`, `KEY_C`, `KEY_D`, `KEY_L1`, `KEY_R1`, `KEY_RUN`, `KEY_MENU` (préfixées
`gb_buttons::`).

### « Maintenu » (niveau) vs « vient d'être enfoncé » (front)

C'est LE concept du chapitre. Imagine qu'on tienne A appuyé pendant 5 images :

```
image :        1     2     3     4     5     6
touche A :    (-)  appui  appui appui appui  (-)
state()   :    0    1     1     1     1     0     ← "maintenu" : vrai tant qu'on appuie
pressed() :    0    1     0     0     0     0     ← "front" : vrai UNE fois, à l'appui
released():    0    0     0     0     0     1     ← "front" : vrai UNE fois, au relâcher
```

- Pour un **déplacement continu** (bouger la raquette tant qu'on pousse), on veut
  **`state()`** / le maintien.
- Pour une **action unique** (lancer la balle, valider un menu, tirer), on veut
  **`pressed()`** / le front — sinon l'action se répéterait à chaque image tant que la
  touche reste enfoncée.

> 🐛 Bug typique : lancer la balle sur le **maintien** de A. Comme A est déjà tenu depuis
> le menu, la balle part instantanément et le joueur n'a rien vu venir. En lançant sur
> le **front** (`pressed`), il faut relâcher puis re-presser : le comportement devient
> correct.

---

## Le joystick

Le joystick donne deux valeurs, de **-1000 à +1000**, autour de 0 au repos :

```cpp
int jx = gb.joystick.get_x();   // gauche négatif  ← 0 →  droite positif
int jy = gb.joystick.get_y();   // haut négatif    ↑ 0 ↓  bas positif
```

Un joystick physique n'est jamais parfaitement à 0 au repos (il « bruite » un peu). Pour
éviter que la raquette dérive toute seule, on ignore les petites valeurs : c'est la
**zone morte**.

```cpp
constexpr int ZONE_MORTE = 200;      // on ignore |jx| < 200
if (jx >  ZONE_MORTE) { /* aller à droite */ }
if (jx < -ZONE_MORTE) { /* aller à gauche */ }
```

```
   -1000        -200     0     +200         +1000
     |------------|-------|-------|------------|
        gauche     ← zone morte →     droite
                  (on n'agit pas)
```

---

## Regrouper proprement

Pour ne pas réécrire tout ça partout, on range l'état lu dans une petite **structure**
`Keys` (une structure = un paquet de variables regroupées sous un nom ; on détaille la
notion au chapitre 8) :

On la déclare **une fois pour toutes** avec tous les champs dont on aura besoin dans la
suite du tutoriel (déplacement au maintien ; actions et navigation au front) :

```cpp
struct Keys {
    // maintien (déplacement continu)
    bool left, right, run, menu;
    // fronts (actions / navigation de menu : une seule image)
    bool a_press, b_press;
    bool up_press, down_press, left_press, right_press;
    // joystick brut
    int  jx, jy;
};

void read_input(Keys& k) {
    gb.pool();                                   // LA lecture unique
    uint16_t s = gb.buttons.state();             // l'état "maintenu" de toutes les touches

    // maintien
    k.left  = s & gb_buttons::KEY_LEFT;
    k.right = s & gb_buttons::KEY_RIGHT;
    k.run   = s & gb_buttons::KEY_RUN;           // servira au retour loader (ch. 21)
    k.menu  = s & gb_buttons::KEY_MENU;          // servira au menu pause (ch. 19)

    // fronts
    k.a_press     = gb.buttons.pressed(gb_buttons::KEY_A);
    k.b_press     = gb.buttons.pressed(gb_buttons::KEY_B);
    k.up_press    = gb.buttons.pressed(gb_buttons::KEY_UP);
    k.down_press  = gb.buttons.pressed(gb_buttons::KEY_DOWN);
    k.left_press  = gb.buttons.pressed(gb_buttons::KEY_LEFT);
    k.right_press = gb.buttons.pressed(gb_buttons::KEY_RIGHT);

    // joystick
    k.jx = gb.joystick.get_x();
    k.jy = gb.joystick.get_y();
}
```

> On remplit `Keys` **en entier** dès maintenant, même si les premiers chapitres n'en
> utilisent qu'une partie : ainsi le menu (ch. 19) et le retour au loader (ch. 21)
> disposeront de `up_press`, `menu`, `run`… sans avoir à revenir modifier cette fonction.

Le `Keys& k` (avec le `&`) veut dire « on te passe la **vraie** structure, pas une
copie » : la fonction la remplit et l'appelant récupère les valeurs. (On reverra les
références plus tard ; pour l'instant, retiens que `&` évite une copie et permet de
modifier l'original.)

**À tester :** affiche `k.jx`/`k.jy` avec `gfx.printf` et bouge le stick ; appuie sur A
et vérifie que `k.a_press` ne s'allume qu'**un instant**, pas tant que tu tiens A.

---

## À retenir

- **Un seul `gb.pool()` par image** ; tout le monde lit le résultat.
- **Maintien** (`state`) pour bouger en continu ; **front** (`pressed`/`released`) pour
  une action unique.
- Le joystick va de **-1000 à +1000** ; ajoute une **zone morte** contre la dérive.

---

[« Précédent](Chapitre_06.md) | [Accueil](index.md) | [Suivant » : La raquette](Chapitre_08.md)
