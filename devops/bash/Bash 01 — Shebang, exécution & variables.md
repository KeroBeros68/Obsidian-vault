#shell #bash #fondamentaux

## Le shebang

La première ligne d'un script indique quel interpréteur doit l'exécuter.

```bash
#!/usr/bin/env bash
```

`env bash` recherche `bash` dans le `PATH` de l'utilisateur, plutôt que de supposer un chemin fixe (`#!/bin/bash`). Plus portable entre systèmes où Bash n'est pas toujours à `/bin/bash` (macOS avec Homebrew, par exemple).

## Rendre un script exécutable

```bash
chmod +x script.sh    # ajoute le droit d'exécution
./script.sh             # exécute le script
bash script.sh           # alternative : exécute sans droit +x nécessaire
```

## Déclarer une variable

```bash
nom="Alice"        # pas d'espace autour du =
age=30
chemin="/home/$nom"   # interpolation dans une chaîne double-quotée
```

Une erreur fréquente : mettre des espaces autour du `=` (`nom = "Alice"`) — Bash interprète alors `nom` comme une commande à exécuter avec des arguments `=` et `"Alice"`, et échoue.

## Lire une variable

```bash
echo "$nom"      # ✅ recommandé
echo $nom         # ⚠️ fonctionne ici, mais dangereux si la valeur contient des espaces
echo "${nom}"     # accolades : utile pour délimiter dans une chaîne ("${nom}_suffix")
```

## Variables locales vs environnement

| Type | Portée | Déclaration |
|------|--------|-------------|
| Variable locale au script | Le script courant et ses sous-shells `()` | `nom="valeur"` |
| Variable d'environnement | Héritée par tous les processus enfants (autres programmes lancés) | `export NOM="valeur"` |

```bash
nom="Alice"           # visible uniquement dans ce script
export NOM="Alice"     # visible aussi par les programmes lancés depuis ce script
```

## Cas particuliers

> [!warning] Toujours quoter une variable utilisée
> `$variable` sans guillemets subit le *word splitting* : si la valeur contient des espaces, Bash la découpe en plusieurs arguments. `"$variable"` protège contre ce comportement.

> [!tip] Variables en lecture seule
> `readonly nom="Alice"` empêche toute modification ultérieure de la variable — utile pour des constantes de script qui ne doivent jamais changer accidentellement.

> [!info] Variables d'environnement courantes
> `$HOME`, `$PATH`, `$USER`, `$PWD` sont définies par le système avant même l'exécution du script.
