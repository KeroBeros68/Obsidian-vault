#shell #bash #paramètres #arguments

## Vue d'ensemble

| Paramètre | Signification |
|-----------|----------------|
| `$0` | Nom du script (ou de la fonction parente) |
| `$1`, `$2`, ... | Arguments positionnels (1er, 2e argument...) |
| `$#` | Nombre d'arguments passés |
| `$@` | Tous les arguments, comme une liste de mots séparés |
| `$*` | Tous les arguments, comme une seule chaîne |
| `$?` | Code de sortie de la dernière commande exécutée |
| `$$` | PID (identifiant de processus) du script courant |

## $0, $1, $#

```bash
#!/usr/bin/env bash
echo "Script : $0"
echo "Premier argument : $1"
echo "Nombre d'arguments : $#"
```

```bash
./script.sh fichier1.txt fichier2.txt
# Script : ./script.sh
# Premier argument : fichier1.txt
# Nombre d'arguments : 2
```

## $@ vs $*

Non quotés, `$@` et `$*` se comportent **de façon identique** : ils s'étendent en liste de mots et subissent le *word splitting*. La différence n'apparaît que **quotés**.

```bash
afficher() {
    for arg in "$@"; do
        echo "Argument : $arg"
    done
}

afficher "fichier un.txt" "fichier deux.txt"
```

Avec `"$@"` — chaque argument original est préservé comme une unité distincte, même s'il contient des espaces :
```
Argument : fichier un.txt
Argument : fichier deux.txt
```

Avec `"$*"` — tous les arguments sont fusionnés en une seule chaîne (séparée par le premier caractère de `$IFS`, l'espace par défaut) :
```
Argument : fichier un.txt fichier deux.txt
```

## $?  — code de sortie

```bash
ls /chemin/inexistant
echo "Code de sortie : $?"   # non nul (échec)

ls /home
echo "Code de sortie : $?"   # 0 (succès)
```

`$?` est écrasé à chaque nouvelle commande — il faut le capturer immédiatement après la commande dont on veut connaître le résultat.

```bash
if [[ $? -eq 0 ]]; then
    echo "Commande réussie"
fi
```

## Cas particuliers

> [!warning] Toujours "$@", jamais $@ ni $*
> Pour itérer sur des arguments en préservant leur intégrité (espaces compris), `"$@"` quoté est la seule forme sûre. `$@` et `$*` non quotés cassent sur tout argument contenant un espace.

> [!tip] $? immédiatement après la commande
> Toute commande intermédiaire (même un simple `echo` de debug) entre la commande testée et la lecture de `$?` écrase sa valeur. Tester `$?` juste après la commande concernée, ou utiliser directement `if commande; then`.

> [!info] $$ pour des fichiers temporaires uniques
> `$$` (PID du script) est parfois utilisé pour générer des noms de fichiers temporaires uniques (`/tmp/script_$$.tmp`), bien que `mktemp` soit l'approche plus robuste et recommandée aujourd'hui.
