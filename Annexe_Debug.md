#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
png2sprite.py - Convertit une image PNG en fichier .h pour la Gamebuino AKA.

Le sprite est embarque comme un tableau C de couleurs 16 bits, au format de l'ecran AKA :
    valeur = (rouge>>3) | ((vert>>2)<<5) | ((bleu>>3)<<11)   # rouge en bits de poids faible

La transparence est encodee par la couleur-cle MAGENTA 0xF81F : tout pixel transparent
(canal alpha < seuil) OU deja magenta pur devient 0xF81F, que le blit du jeu ignore.

Usage :
    python3 png2sprite.py hero.png
    python3 png2sprite.py hero.png --name hero --out hero.h
    python3 png2sprite.py hero.png --alpha-threshold 128

Dependances : Pillow  (pip install Pillow)
"""
import argparse
import os
import sys

try:
    from PIL import Image
except ImportError:
    sys.exit("Erreur : Pillow manquant. Installe-le avec : pip install Pillow")

MAGENTA_KEY = 0xF81F  # couleur-cle de transparence (rouge+bleu a fond, vert nul)


def to_aka565(r, g, b):
    """RGB 8 bits -> couleur 16 bits AKA (rouge en bits de poids faible)."""
    return (r >> 3) | ((g >> 2) << 5) | ((b >> 3) << 11)


def sanitize_name(name):
    """Rend un nom compatible avec une variable C (lettres, chiffres, _)."""
    out = "".join(c if (c.isalnum() or c == "_") else "_" for c in name)
    if not out or out[0].isdigit():
        out = "sprite_" + out
    return out


def convert(path, name=None, alpha_threshold=128):
    img = Image.open(path).convert("RGBA")   # on force RGBA pour lire l'alpha
    w, h = img.size
    px = img.load()

    if name is None:
        name = sanitize_name(os.path.splitext(os.path.basename(path))[0])

    values = []
    for y in range(h):
        for x in range(w):
            r, g, b, a = px[x, y]
            if a < alpha_threshold:
                values.append(MAGENTA_KEY)                # transparent -> cle
            elif (r, g, b) == (255, 0, 255):
                values.append(MAGENTA_KEY)                # magenta pur -> cle aussi
            else:
                values.append(to_aka565(r, g, b))
    return name, w, h, values


def emit_header(name, w, h, values, out_path):
    lines = []
    lines.append("// Fichier genere par png2sprite.py - ne pas editer a la main.")
    lines.append("#pragma once")
    lines.append("#include <stdint.h>")
    lines.append("")
    lines.append(f"constexpr int  {name}_W = {w};")
    lines.append(f"constexpr int  {name}_H = {h};")
    lines.append(f"constexpr uint16_t {name}_KEY = 0x{MAGENTA_KEY:04X}; // couleur transparente")
    lines.append("")
    lines.append(f"const uint16_t {name}[{w} * {h}] = {{")
    # une ligne par rangee de l'image, pour rester lisible
    for y in range(h):
        row = values[y * w:(y + 1) * w]
        cells = ", ".join(f"0x{v:04X}" for v in row)
        lines.append(f"    {cells},")
    lines.append("};")
    lines.append("")
    with open(out_path, "w", encoding="utf-8") as f:
        f.write("\n".join(lines))


def main():
    ap = argparse.ArgumentParser(description="Convertit un PNG en sprite .h pour la Gamebuino AKA.")
    ap.add_argument("png", help="image d'entree (.png)")
    ap.add_argument("--name", help="nom de la variable C (defaut : nom du fichier)")
    ap.add_argument("--out", help="fichier .h de sortie (defaut : <name>.h)")
    ap.add_argument("--alpha-threshold", type=int, default=128,
                    help="seuil d'opacite 0..255 en dessous duquel un pixel est transparent (defaut 128)")
    args = ap.parse_args()

    name, w, h, values = convert(args.png, args.name, args.alpha_threshold)
    out = args.out or (name + ".h")
    emit_header(name, w, h, values, out)
    print(f"OK : {args.png}  ->  {out}   ({w}x{h}, {len(values)} pixels, variable '{name}')")


if __name__ == "__main__":
    main()
