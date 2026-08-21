#dev #regex #bash

## grep — rechercher des motifs

```bash
# Recherche simple
grep "motif" fichier.txt

# Expressions régulières étendues (ERE) — nécessaire pour {}, +, ?, |
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" fichier.txt
# équivalent : egrep "..." fichier.txt

# Insensible à la casse
grep -i "motif" fichier.txt

# Recherche récursive dans un répertoire
grep -r "motif" /chemin/vers/repertoire
```

Par défaut, `grep` utilise les expressions régulières basiques (BRE), où `{}`, `+`, `?`, `|` doivent être échappés (`\+`, `\?`...) pour être interprétés comme métacaractères. L'option `-E` (ou `egrep`) active les expressions régulières étendues (ERE), plus proches de la syntaxe présentée dans [[Regex 01 — Syntaxe et métacaractères]].

## sed — modifier du texte à la volée

```bash
# Remplacer la première occurrence par ligne
sed 's/motif/remplacement/' fichier.txt

# Remplacer TOUTES les occurrences par ligne (flag g = global)
sed 's/motif/remplacement/g' fichier.txt

# Substitution avancée avec ERE (-E), ex. anonymiser des e-mails
sed -E 's/[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/[email protected]/g' fichier.txt

# Modifier le fichier directement (sans redirection)
sed -i 's/motif/remplacement/g' fichier.txt

# Supprimer les lignes correspondant à un motif
sed '/motif/d' fichier.txt
```

> [!warning] `-i` modifie le fichier en place
> Sans sauvegarde, la modification est irréversible. Tester d'abord sans `-i` (la sortie va vers stdout) ou utiliser `sed -i.bak` pour conserver une copie.

## awk — manipuler des données structurées

```bash
# Imprimer les lignes correspondant à un motif ($0 = ligne entière)
awk '/motif/ {print $0}' fichier.txt

# Extraire un champ spécifique (-F définit le séparateur, $2 = 2e champ)
awk -F, '/motif/ {print $2}' fichier.csv

# Extraire des adresses IP d'un fichier log
awk '/[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/ {print $1}' fichier.log
```

`awk` combine recherche par regex et traitement par champs — utile pour les fichiers structurés (CSV, logs) où `grep`/`sed` ne suffisent pas à isoler une colonne.

## Combiner les outils

```bash
# Rechercher récursivement les fichiers contenant une adresse e-mail,
# puis remplacer cette adresse dans chacun d'eux
grep -rl "exemple@domaine.com" /chemin/vers/repertoire | \
  xargs sed -i 's/exemple@domaine.com/[email protected]/g'
```

`grep -rl` liste les fichiers concernés (sans afficher les lignes), `xargs` les transmet un à un à `sed -i` pour la substitution.

## Prérequis & suite

- [[Regex — Index des fiches]] ← retour à l'index du module
- [[Regex 01 — Syntaxe et métacaractères]] ← prérequis : syntaxe des motifs utilisés ici
- [[Bash — Index des fiches]] ← scripting Bash plus large (au-delà du traitement de texte)
- [[Regex 03 — Le module re en Python]] ← les mêmes motifs appliqués en Python
