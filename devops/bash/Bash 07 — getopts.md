#shell #bash #getopts #cli

## Pourquoi getopts

Pour un script appelé avec des options façon commande Unix (`-v`, `-f fichier.txt`), parser `$1`, `$2`... manuellement devient vite source d'erreurs. `getopts` est l'outil intégré à Bash (POSIX) pour structurer ce parsing.

## Syntaxe de base

```bash
#!/usr/bin/env bash

verbose=0
fichier=""

while getopts ":vf:h" opt; do
    case "$opt" in
        v) verbose=1 ;;
        f) fichier="$OPTARG" ;;
        h) echo "Usage: $0 [-v] [-f fichier] [-h]"; exit 0 ;;
        \?) echo "Option invalide : -$OPTARG" >&2; exit 1 ;;
        :) echo "L'option -$OPTARG nécessite un argument." >&2; exit 1 ;;
    esac
done
shift $((OPTIND - 1))   # retire les options déjà traitées, laisse les arguments positionnels

echo "verbose=$verbose fichier=$fichier reste=$*"
```

## Anatomie de la chaîne d'options

```
":vf:h"
 │ │  │
 │ │  └─ -h : option simple, pas d'argument
 │ └──── -f : le ':' après signifie qu'elle ATTEND un argument (récupéré dans $OPTARG)
 └────── ':' en tête : active le mode d'erreur silencieux (gestion manuelle des erreurs)
```

| Caractère | Effet |
|-----------|-------|
| Lettre seule (`v`) | Option sans argument (flag/switch) |
| Lettre suivie de `:` (`f:`) | Option qui attend un argument, accessible via `$OPTARG` |
| `:` en première position | Désactive les messages d'erreur automatiques de getopts, au profit d'une gestion personnalisée via `\?` et `:` |

## OPTIND et shift

`OPTIND` est automatiquement incrémenté par `getopts` à chaque option traitée. Après la boucle, `shift $((OPTIND - 1))` retire toutes les options déjà consommées — ce qui reste dans `$@` correspond aux arguments positionnels (ex. noms de fichiers passés sans `-`).

```bash
./script.sh -v -f config.yml fichier1.txt fichier2.txt
# Après la boucle getopts : $@ contient fichier1.txt fichier2.txt
```

## Limite : pas d'options longues natives

`getopts` ne gère que des options courtes (`-v`), pas les options longues façon `--verbose`. Pour les supporter, deux approches courantes :

```bash
# Option A : utiliser la commande externe getopt (GNU), qui supporte les deux
args=$(getopt -o vf: --long verbose,fichier: -- "$@")

# Option B : boucle manuelle avant de passer le relais à getopts pour les options courtes
case "$1" in
    --verbose) verbose=1; shift ;;
    --fichier) fichier="$2"; shift 2 ;;
esac
```

## Cas particuliers

> [!warning] getopts ne comprend pas --option
> Une option longue passée à un script utilisant `getopts` seul est traitée comme un argument positionnel ordinaire, pas comme une option — source de bugs silencieux si l'utilisateur du script s'attend à du style GNU classique.

> [!tip] Toujours commencer la chaîne par :
> Démarrer la chaîne d'options par `:` (`":vf:h"`) donne un contrôle total sur les messages d'erreur via les cas `\?` (option inconnue) et `:` (argument manquant), plutôt que de subir les messages génériques de Bash.

> [!info] getopts vs getopt
> `getopts` est un builtin Bash/POSIX, portable et toujours disponible. `getopt` (sans s) est une commande externe, plus puissante (options longues natives) mais pas garantie présente sur tous les systèmes (notamment certains environnements minimalistes ou macOS par défaut).
