# Chapitre 20 — Optimiser (quand c'est utile)

[« Précédent](Chapitre_19.md) | [Accueil](index.md) | [Suivant »](Chapitre_21.md)

![Niveau](https://img.shields.io/badge/Niveau-Avanc%C3%A9-red)

---

## Objectif

Rendre le jeu plus fluide **si besoin**. Règle d'or d'abord : **on n'optimise pas à
l'aveugle**. On mesure, on trouve ce qui coûte, on corrige ce point précis. Optimiser du
code qui n'est pas le goulot d'étranglement, c'est du temps perdu et des bugs en plus.

---

## D'abord : mesurer

La lib donne la cadence réelle. Affiche-la et observe **quand** elle chute :

```cpp
gfx.setColor(color_white);
gfx.move_cursor(2, 2);
gfx.printf("FPS %.0f", gfx.get_fps());
```

Si les FPS s'effondrent quand il y a beaucoup de briques/particules, c'est **là** qu'il
faut regarder. S'ils sont stables, **ne touche à rien**.

---

## Les optimisations qui rapportent vraiment

### 1. Préférer les fonctions de formes au pixel-par-pixel

On l'a vu au chapitre 4 : `gfx.fillRect(...)` remplit une zone bien plus vite qu'une
double boucle de `drawPixel`. Pour de grandes surfaces, une seule fonction de forme vaut
des milliers d'appels individuels. **Utilise les formes de la lib** pour tout ce qui est
rectangle/ligne/cercle.

### 2. Réserver la mémoire des `vector` une fois pour toutes

Quand un `vector` grandit, il peut devoir **réallouer** (recopier) son contenu — coûteux
si ça arrive en plein jeu. Si tu connais un maximum raisonnable, réserve-le **au
démarrage** :

```cpp
bricks.reserve(64);      // place pour 64 briques, allouée une seule fois
shots.reserve(16);
falling.reserve(16);
```

L'idée générale : **éviter d'allouer/libérer de la mémoire pendant le jeu**. On prépare
les réserves à l'init, on réutilise ensuite.

### 3. Ne pas recalculer ce qui ne change pas

Sors des boucles les valeurs constantes. Par exemple, si tu utilises souvent
`p.r.w / 2`, calcule-le une fois dans une variable plutôt qu'à chaque itération. Un calcul
trivial une fois par image ne coûte rien ; le **même** calcul répété des milliers de fois
par image, si.

### 4. L'audio sur son cœur, le jeu sur le sien

On l'a fait au chapitre 13 : la tâche audio tourne sur le **cœur 0**, la boucle de jeu
sur l'autre. Ça évite que les gros calculs du jeu (beaucoup de collisions) ne fassent
« bégayer » le son. Si tu ajoutes une IA lourde (jeu de plateau, etc.), même logique :
mets-la sur une tâche à part.

---

## Ce qui est déjà géré pour toi

- **Le découpage aux bords** (*clipping*) : `gfx.drawPixel` vérifie déjà les bornes de
  l'écran (chapitre 4). Tu peux dessiner un objet qui dépasse : les pixels hors écran sont
  simplement ignorés. Pas besoin de le gérer toi-même pour commencer.
- **Le transfert vers l'écran** : `gfx.update()` envoie le framebuffer à l'écran de façon
  efficace (transfert en bloc). Tu appelles `update()` **une fois** par image, pas plus.

---

## Fausses bonnes idées (pièges)

- **« Effacer seulement ce qui a bougé »** : tentant, mais source de traînées et de bugs.
  Sur un casse-briques, redessiner tout l'écran chaque image est assez rapide. Ne
  complexifie que si la mesure le justifie.
- **« Micro-optimiser chaque ligne »** : illisible, risqué, et souvent inutile. Le
  compilateur optimise déjà énormément. Vise la **clarté** d'abord.
- **Optimiser avant de mesurer** : tu risques de durcir un code qui n'était pas le
  problème.

---

## À retenir

- **Mesure d'abord** (`get_fps`), optimise **ensuite**, et seulement le point qui coûte.
- Gains réels : **formes de la lib** plutôt que pixel-par-pixel, **`reserve()`** sur les
  vectors, **pas d'allocation en jeu**, audio/jeu **sur des cœurs séparés**.
- Le **clipping** et le **transfert écran** sont déjà gérés par la lib.

---

[« Précédent](Chapitre_19.md) | [Accueil](index.md) | [Suivant » : Assemblage final](Chapitre_21.md)
