#shell #bash #chaînes #avancé

## Command substitution : $(...)

```bash
date_du_jour=$(date +%Y-%m-%d)
nombre_fichiers=$(ls | wc -l)
echo "Nous sommes le $date_du_jour, $nombre_fichiers fichiers ici"
```

`$(...)` exécute la commande et capture sa sortie dans une variable. Préféré aux backticks `` `...` `` (syntaxe historique, toujours valide mais moins lisible et imbriquée avec difficulté).

```bash
# ✅ Lisible et imbriquable facilement
resultat=$(echo $(date +%Y))

# ⚠️ Backticks : fonctionne, mais syntaxe historique à éviter dans du code neuf
resultat=`date +%Y`
```

## Arithmétique : $(( ))

```bash
a=5
b=3
somme=$((a + b))
echo "$somme"   # 8

# Opérateurs disponibles : + - * / % ** (puissance)
echo $((10 % 3))    # 1 (modulo)
echo $((2 ** 8))     # 256
```

## Expansion de paramètres : retirer un motif

```bash
fichier="rapport.tar.gz"

echo "${fichier%.gz}"      # rapport.tar       — retire le plus court motif depuis la FIN
echo "${fichier%%.*}"       # rapport            — retire le plus long motif depuis la FIN
echo "${fichier#*.}"        # tar.gz             — retire le plus court motif depuis le DÉBUT
echo "${fichier##*.}"       # gz                  — retire le plus long motif depuis le DÉBUT
```

| Syntaxe | Direction | Longueur du motif retiré |
|---------|-----------|---------------------------|
| `${var#motif}` | Depuis le début | Le plus court possible |
| `${var##motif}` | Depuis le début | Le plus long possible |
| `${var%motif}` | Depuis la fin | Le plus court possible |
| `${var%%motif}` | Depuis la fin | Le plus long possible |

Pattern très utilisé pour extraire une extension de fichier (`${fichier##*.}`) ou un nom sans extension (`${fichier%.*}`).

## Substitution de motif

```bash
chemin="/home/user/documents"
echo "${chemin/user/alice}"     # remplace la PREMIÈRE occurrence : /home/alice/documents
echo "${chemin//o/0}"            # remplace TOUTES les occurrences : /h0me/user/d0cuments
```

## Valeurs par défaut

```bash
echo "${nom:-Inconnu}"     # affiche "Inconnu" si $nom est vide ou non défini, sans le modifier
echo "${nom:=Inconnu}"      # idem, MAIS assigne aussi "Inconnu" à $nom si vide
```

Pattern courant pour des scripts avec arguments optionnels : `dossier="${1:-/tmp}"` utilise `/tmp` si aucun argument n'est passé.

## Longueur d'une chaîne

```bash
texte="bonjour"
echo "${#texte}"   # 7
```

## Cas particuliers

> [!warning] $(( )) vs [[ ]] pour l'arithmétique
> `$(( ))` retourne une **valeur** (à utiliser dans une assignation ou un affichage), tandis que `(( ))` seul (sans `$`) retourne un **statut de succès/échec** utilisable directement dans un `if` — ne pas confondre les deux usages.

> [!tip] ${var:-defaut} pour des scripts robustes
> Utiliser systématiquement `${1:-valeur_par_defaut}` pour les arguments optionnels évite d'avoir à écrire des `if [[ -z "$1" ]]` répétitifs pour chaque paramètre.

> [!info] % et # se retiennent par leur position
> Astuce mnémotechnique : `#` est à gauche du clavier numérique (pense "début"), `%` évoque davantage une fin de calcul — mais le plus fiable reste de retenir que `#` agit depuis le **d**ébut et `%` depuis la fin.
