#shell #bash #fondamentaux

## if / elif / else

```bash
if [ "$age" -ge 18 ]; then
    echo "Majeur"
elif [ "$age" -ge 13 ]; then
    echo "Adolescent"
else
    echo "Enfant"
fi
```

## [ ] vs [[ ]]

```bash
# [ ] — commande de test POSIX, portable, plus de pièges
if [ "$nom" = "Alice" ]; then echo "ok"; fi

# [[ ]] — extension Bash, plus sûre et plus expressive
if [[ "$nom" == "Alice" ]]; then echo "ok"; fi
```

| Aspect | `[ ]` | `[[ ]]` |
|--------|-------|---------|
| Portabilité | POSIX, fonctionne dans `sh` | Spécifique à Bash/ksh/zsh |
| Word splitting sur variables non quotées | Risque réel | Désactivé automatiquement |
| Comparaison de chaînes avec motifs (`==`, wildcards) | Non | Oui |
| Opérateurs logiques internes (`&&`, `\|\|`) | Syntaxe lourde (`-a`, `-o`) | `&&`, `\|\|` directement |

```bash
# ❌ Risqué avec [ ] si $var contient des espaces non quotés
[ $var = "test" ]

# ✅ [[ ]] protège même sans quotes (mais quoter reste la bonne pratique)
[[ $var == "test" ]]
```

## Tests de fichiers courants

| Test | Signification |
|------|----------------|
| `-f fichier` | Existe et est un fichier régulier |
| `-d dossier` | Existe et est un dossier |
| `-e chemin` | Existe (fichier ou dossier) |
| `-r fichier` | Lisible |
| `-w fichier` | Inscriptible |
| `-x fichier` | Exécutable |
| `-s fichier` | Existe et n'est pas vide |

```bash
if [[ -f "config.yml" ]]; then
    echo "Le fichier de config existe"
fi
```

## case

```bash
case "$1" in
    start)
        echo "Démarrage"
        ;;
    stop)
        echo "Arrêt"
        ;;
    restart|reload)
        echo "Redémarrage"
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

`case` est souvent plus lisible qu'une longue chaîne de `if`/`elif` quand on teste une seule variable contre plusieurs valeurs possibles — pattern courant dans les scripts de gestion de service (`start`/`stop`/`restart`).

## Cas particuliers

> [!warning] = vs == dans [ ]
> Dans `[ ]` POSIX, seul `=` est garanti pour la comparaison de chaînes (`==` fonctionne en Bash mais n'est pas portable). Dans `[[ ]]`, `==` et `=` sont équivalents.

> [!tip] Toujours quoter, même avec [[ ]]
> Bien que `[[ ]]` désactive le word splitting, garder l'habitude de quoter (`"$var"`) évite les surprises si on bascule un jour entre `[ ]` et `[[ ]]`, et reste plus lisible.

> [!info] [[ ]] côté droit d'une comparaison
> À droite d'un `==` dans `[[ ]]`, une chaîne non quotée est interprétée comme un motif (wildcards `*`, `?`) plutôt que comme une chaîne littérale. Quoter le côté droit si on veut une comparaison exacte.
