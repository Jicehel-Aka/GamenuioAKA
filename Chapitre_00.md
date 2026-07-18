# Chapitre 00 — Notions de C++ pour démarrer

[Accueil](index.md) | [Chapitre suivant »](Chapitre_01.md)

![Niveau](https://img.shields.io/badge/Niveau-D%C3%A9butant-green)

---

## À quoi sert ce chapitre

Le tutoriel n'exige **aucune** connaissance préalable du C++. Ce chapitre pose, en une
page, le vocabulaire minimal pour lire les suivants sans buter. Tu peux le survoler et y
**revenir** quand un mot te surprend (chaque notion renvoie au chapitre qui l'utilise).

Ce n'est **pas** un cours complet de C++ : juste ce qu'il faut pour la AKA.

---

## Variables et types

Une **variable** est une case mémoire à laquelle on donne un nom et un **type** (la nature
de ce qu'elle contient) :

```cpp
int   score = 0;        // un entier (nombre sans virgule)
float vitesse = 2.6f;   // un nombre à virgule (le 'f' = float)
bool  en_jeu = true;    // vrai / faux (true / false)
char  lettre = 'A';     // un caractère
```

### Pourquoi des types comme `uint16_t` et pas juste `int` ?

Sur une petite puce, **la taille compte**. On utilise donc des types à **taille fixe et
connue**, dont le nom dit tout : `u` = *unsigned* (non signé, positif ou nul), le nombre
= les bits.

| Type | Taille | Contient | Va de… à… |
|---|---|---|---|
| `uint8_t`  | 8 bits (1 octet) | entier non signé | 0 à 255 |
| `uint16_t` | 16 bits (2 octets)| entier non signé | 0 à 65 535 |
| `uint32_t` | 32 bits (4 octets)| entier non signé | 0 à ~4 milliards |
| `int16_t` / `int32_t` | 16 / 32 bits | entier **signé** (peut être négatif) | ± moitié |

- **Non signé** (`unsigned`) = jamais négatif. Une couleur, un masque de touches, une
  position d'écran : ça n'a pas de sens d'être négatif, donc on prend du non signé.
- Une **couleur** de pixel tient sur `uint16_t` (16 bits) — voir [chapitre 4](Chapitre_04.md).
- Le **paquet de boutons** est un `uint16_t` où chaque bit est une touche — voir
  [chapitre 7](Chapitre_07.md).

---

## Les bits, et le « et » bit à bit (`&`)

Un nombre est, en mémoire, une suite de **bits** (des 0 et des 1). On peut s'en servir
comme d'une rangée d'interrupteurs : chaque bit = un oui/non. C'est ainsi que la lib range
l'état des boutons dans **un seul** nombre.

```
 valeur des boutons :  0 0 0 0 0 0 0 1 0 0 1 0 0 0 0 0
                                     A            LEFT      (exemple)
```

Pour savoir si **A** est enfoncée, on isole **son** bit avec l'opérateur **`&`** (« et »
bit à bit) et le masque `KEY_A` (un nombre qui n'a que le bit de A à 1) :

```cpp
if (etat & gb_buttons::KEY_A) { /* A est enfoncée */ }
```

Le `&` met à 0 tous les bits sauf celui qu'on teste : le résultat n'est « vrai » que si ce
bit précis était à 1. On appelle ça un **masque**. (Utilisé au [chapitre 7](Chapitre_07.md).)

> Ne confonds pas `&` (un seul, bit à bit) et `&&` (« et » logique entre deux conditions,
> comme dans `if (a && b)`).

---

## `const` et `constexpr`

- **`const`** : « cette valeur ne changera pas ». Protège d'une modification accidentelle.
- **`constexpr`** : une constante **connue dès la compilation**, à laquelle on donne un
  nom lisible. On s'en sert pour les réglages :

```cpp
constexpr int SCREEN_W = 320;     // au lieu d'écrire "320" partout
constexpr int PADDLE_SPEED = 5;   // changer la difficulté = changer ce nombre
```

---

## Les fonctions

Une **fonction** est un bloc de code réutilisable, avec des **paramètres** (ce qu'on lui
donne) et éventuellement une **valeur de retour** (ce qu'elle rend). `void` = « ne rend
rien ».

```cpp
int add(int a, int b) {   // prend deux entiers, en rend un
    return a + b;         // la valeur rendue
}

void efface() {           // ne rend rien
    gfx.clear(color_black);
}
```

---

## `#include`, en-tête et source

**`#include "..."`** recopie le contenu d'un autre fichier à cet endroit, **avant** la
compilation. On s'en sert pour rendre disponibles des fonctions/types définis ailleurs :

```cpp
#include "gamebuino.h"    // rend accessible toute l'API de la lib AKA
```

Un projet se répartit entre **en-têtes** (`.h` : les *déclarations* — « voici ce qui
existe ») et **sources** (`.cpp` : le *code* — « voici comment ça marche »). On
`#include` un `.h` pour utiliser ce qu'il annonce. (Découpage détaillé au
[chapitre 21](Chapitre_21.md).)

---

## Structures et objets

Une **structure** (`struct`) regroupe plusieurs variables sous un même nom. On accède à
un champ avec un **point** :

```cpp
struct Rect { int x, y, w, h; };   // un rectangle = 4 nombres liés
Rect r;
r.x = 10;  r.w = 48;               // accès avec le point
```

Un **objet** est la même idée, en plus riche : des données **et** des fonctions
regroupées. La lib t'en fournit deux tout prêts : `gb` (la console) et `gfx` (le dessin).
On les utilise exactement comme des structures, avec le point, sauf qu'on **appelle** aussi
leurs fonctions :

```cpp
gb.init();               // on appelle une fonction de l'objet gb
gfx.fillRect(0,0,10,10); // et une de l'objet gfx
```

Pas besoin d'en savoir plus sur les « classes » pour ce tutoriel : structures pour **tes**
données, objets `gb`/`gfx` pour parler à la console. (Structures détaillées au
[chapitre 8](Chapitre_08.md).)

---

## Références (`&`) et pointeurs (`*`)

Quand on passe une variable à une fonction, C++ en fait normalement une **copie**. Deux
outils permettent d'éviter la copie (utile pour les grosses données) ou de **modifier
l'original** :

- une **référence** `Type& nom` : « l'original, pas une copie ». La fonction peut le
  remplir.

  ```cpp
  void read_input(Keys& k) { ... }   // remplit le vrai k de l'appelant (chapitre 7)
  ```

- un **pointeur** `Type* nom` : contient l'**adresse** d'une donnée. Le nom d'un tableau
  se comporte comme un pointeur vers sa première case — c'est pour ça qu'on passe un
  sprite ainsi :

  ```cpp
  void draw_sprite(const uint16_t* sprite, int w, int h, int x, int y) { ... } // chapitre 4
  ```

Pour débuter, retiens surtout la **référence `&`** (pour que la fonction remplisse ta
structure) et le fait qu'un **tableau se passe via un pointeur `*`**. On n'a pas besoin
d'aller plus loin.

---

## Quelques mots-clés qu'on croisera

- **`auto`** : « déduis le type tout seul ». `for (auto& b : bricks)` = « pour chaque
  brique `b` » sans réécrire le type.
- **`nullptr`** : un pointeur « qui ne pointe sur rien » (aucune adresse).
- **`static`** (dans une fonction) : une variable qui **garde sa valeur** d'un appel à
  l'autre (on l'utilise pour le chrono du combo loader, chapitre 21).

---

## La mémoire est comptée (c'est de l'embarqué !)

Sur un PC, on ne pense presque jamais à la mémoire. Sur l'ESP32-S3, **si**. Quelques
repères pour prendre les bons réflexes dès le départ :

- Le framebuffer (l'écran) occupe déjà **~150 Ko** (320×240×2 octets). C'est énorme à
  cette échelle.
- Un gros sprite coûte cher : chaque pixel = **2 octets** embarqués dans le programme.
  Préfère des sprites petits (8×8 à 24×24) — voir l'[Annexe A](Annexe_Sprites.md).
- Un `std::vector` qui grandit **alloue** de la mémoire ; on **réserve** sa taille au
  démarrage pour éviter les surprises en jeu (chapitre 20).
- On peut surveiller la mémoire libre avec `gb.free_sram()` et `gb.free_psram()`.

Rien d'inquiétant pour un casse-briques : il suffit d'éviter le gaspillage (gros tableaux
inutiles, allocations en pleine boucle).

---

## À retenir

- Types à taille fixe (`uint8_t/16/32`), **non signé** = jamais négatif.
- Les **bits** et le masque **`&`** servent à lire les boutons.
- **`struct`** pour tes données ; **`gb`/`gfx`** sont des **objets** de la lib.
- **`&`** (référence) évite la copie / modifie l'original ; un **tableau** se passe via un
  **pointeur `*`**.
- La **mémoire est limitée** : petits sprites, `reserve()`, pas d'allocation en pleine
  boucle.

---

[Accueil](index.md) | [Chapitre suivant » : Introduction](Chapitre_01.md)
