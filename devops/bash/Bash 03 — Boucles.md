#shell #bash #fondamentaux

## for

```bash
# Itérer sur une liste de valeurs
for fruit in pomme banane cerise; do
    echo "$fruit"
done

# Itérer sur des fichiers (toujours quoter la variable de boucle)
for fichier in *.txt; do
    echo "Traitement de $fichier"
done

# Boucle numérique (style C)
for ((i = 0; i < 5; i++)); do
    echo "Itération $i"
done

# Plage de nombres
for i in {1..5}; do
    echo "$i"
done
```

## while

```bash
compteur=0
while [[ $compteur -lt 5 ]]; do
    echo "Compteur : $compteur"
    ((compteur++))
done

# Lire un fichier ligne par ligne
while IFS= read -r ligne; do
    echo "Ligne : $ligne"
done < fichier.txt
```

## until

```bash
# S'exécute TANT QUE la condition est FAUSSE (inverse de while)
compteur=0
until [[ $compteur -ge 5 ]]; do
    echo "$compteur"
    ((compteur++))
done
```

## break et continue

```bash
for i in {1..10}; do
    if [[ $i -eq 5 ]]; then
        break        # sort complètement de la boucle
    fi
    if (( i % 2 == 0 )); then
        continue      # passe à l'itération suivante
    fi
    echo "$i"
done
```

## Cas particuliers

> [!warning] for fichier in *.txt sans correspondance
> Si aucun fichier `.txt` n'existe, `*.txt` reste littéral (non remplacé) par défaut en Bash — la boucle traite alors la chaîne `*.txt` comme une seule valeur. Activer `shopt -s nullglob` en début de script pour que l'absence de correspondance donne une liste vide à la place.

> [!tip] IFS= read -r, le duo systématique
> `IFS=` empêche la suppression des espaces en début/fin de ligne, `-r` empêche l'interprétation des antislashs comme caractères d'échappement. Combinaison standard pour lire un fichier ligne par ligne sans surprise.

> [!info] (( )) pour l'arithmétique
> `((compteur++))` et `(( i % 2 == 0 ))` utilisent la syntaxe arithmétique de Bash, plus lisible que `[ ]` pour les comparaisons et opérations numériques. Détails dans [[Bash 09 — Manipulation de chaînes & substitutions]].
