#shell #bash #pièges #erreurs #debugging

## 🪤 Piège 1 — Variable non quotée et word splitting

```bash
fichier="mon rapport 2026.csv"
cp $fichier /backup/   # ❌ découpé en 3 arguments distincts, cp échoue
```

```bash
cp "$fichier" /backup/   # ✅ traité comme un seul argument
```

> [!warning] Toujours quoter
> Une variable non quotée contenant un espace est automatiquement découpée en plusieurs mots. Voir [[Bash 01 — Shebang, exécution & variables]].

---

## 🪤 Piège 2 — Espaces autour du = dans une assignation

```bash
nom = "Alice"   # ❌ Bash interprète "nom" comme une commande
nom="Alice"      # ✅ aucun espace autour du =
```

> [!tip] Mémo
> Contrairement à beaucoup de langages, Bash n'accepte aucun espace autour du `=` dans une assignation.

---

## 🪤 Piège 3 — for sur un glob sans correspondance

```bash
for f in *.txt; do
    echo "$f"
done
# Si aucun .txt n'existe : affiche littéralement "*.txt" au lieu de ne rien afficher
```

> [!warning] Activer nullglob
> `shopt -s nullglob` en début de script fait en sorte qu'un glob sans correspondance donne une liste vide plutôt que le motif littéral. Voir [[Bash 03 — Boucles]].

---

## 🪤 Piège 4 — Oublier local dans une fonction

```bash
compteur=10

modifier() {
    compteur=0   # ❌ écrase la variable GLOBALE, pas une copie locale
}
modifier
echo "$compteur"   # 0, alors qu'on ne s'y attendait pas
```

> [!tip] Mémo
> Toute variable assignée dans une fonction est globale par défaut, sauf si déclarée avec `local`. Voir [[Bash 04 — Fonctions]].

---

## 🪤 Piège 5 — 2>&1 placé avant la redirection de stdout

```bash
commande 2>&1 > fichier.txt   # ❌ stderr part toujours vers le terminal
commande > fichier.txt 2>&1   # ✅ stdout ET stderr vont dans le fichier
```

> [!warning] L'ordre compte
> `2>&1` doit toujours venir après la redirection de `stdout` pour que les deux flux finissent au même endroit. Voir [[Bash 06 — Redirections & pipes]].

---

## 🪤 Piège 6 — getopts et les options longues

```bash
./script.sh --verbose   # ❌ getopts seul ne comprend pas --verbose
```

> [!warning] Limite native
> `getopts` ne gère que les options courtes (`-v`). Pour `--verbose`, il faut soit `getopt` externe, soit une boucle manuelle. Voir [[Bash 07 — getopts]].

---

## 🪤 Piège 7 — set -e qui ne se déclenche pas toujours

```bash
set -e
((compteur++))   # ❌ si compteur vaut 0, le code de sortie de (( )) peut interrompre le script
```

> [!warning] set -e a des angles morts
> `set -e` ne se déclenche pas dans certains contextes (commande dans un `if`, suivie de `||`, ou certains cas avec `(( ))`). Voir [[Bash 08 — Scripts robustes]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Variable non quotée cassée par les espaces | Toujours `"$variable"` |
| Espaces autour du `=` | Jamais d'espace dans une assignation |
| Glob sans correspondance affiché littéralement | `shopt -s nullglob` |
| Variable de fonction qui écrase la globale | Toujours `local` dans les fonctions |
| stderr non redirigé malgré `2>&1` | `2>&1` après la redirection de stdout |
| `--option` non reconnue par getopts | `getopt` externe ou boucle manuelle |
| `set -e` qui n'arrête pas le script où on l'attend | Connaître ses angles morts, tester avec `set -x` |
