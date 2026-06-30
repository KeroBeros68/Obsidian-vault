#shell #bash #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Shebang** | Première ligne d'un script (`#!/usr/bin/env bash`) indiquant l'interpréteur à utiliser. |
| **Word splitting** | Découpage automatique d'une valeur non quotée en plusieurs mots, sur la base des espaces (`$IFS`). Source fréquente de bugs si une variable n'est pas quotée. |
| **Quoting** | Usage de guillemets (`"..."`) ou apostrophes (`'...'`) pour contrôler l'interprétation d'une chaîne par le shell. |
| **Paramètre positionnel** | Argument passé à un script ou une fonction, accessible via `$1`, `$2`, etc. |
| **Variable d'environnement** | Variable exportée (`export`), héritée par tous les processus enfants lancés depuis le script. |
| **Scope local** | Portée d'une variable déclarée avec `local` dans une fonction, limitée à cette fonction. |
| **Code de sortie (exit status)** | Entier entre 0 et 255 retourné par une commande ; `0` signifie succès, tout autre code signifie échec. Accessible via `$?`. |
| **getopts** | Builtin Bash/POSIX pour parser des options de ligne de commande courtes (`-v`, `-f valeur`). |
| **OPTARG / OPTIND** | Variables internes de `getopts` : `OPTARG` contient l'argument de l'option courante, `OPTIND` l'index de la prochaine option à traiter. |
| **Redirection** | Mécanisme déviant un flux (`stdin`, `stdout`, `stderr`) vers ou depuis un fichier (`>`, `<`, `2>`). |
| **Pipe** | Connexion de la sortie standard d'une commande vers l'entrée standard d'une autre (`\|`). |
| **Here-document** | Bloc de texte multi-lignes intégré directement dans un script (`<< EOF ... EOF`). |
| **set -e** | Option arrêtant immédiatement le script si une commande échoue (code de sortie non nul). |
| **set -u** | Option arrêtant le script si une variable non définie est utilisée. |
| **pipefail** | Option faisant échouer un pipe entier si n'importe laquelle de ses commandes échoue, pas seulement la dernière. |
| **trap** | Commande exécutant une action donnée lors d'un signal ou événement précis (ex. `EXIT`, `SIGINT`). |
| **Command substitution** | Capture de la sortie d'une commande dans une variable, via `$(...)`. |
| **Expansion de paramètres** | Manipulation de la valeur d'une variable directement dans sa syntaxe d'accès (`${var%motif}`, `${var:-defaut}`...). |
