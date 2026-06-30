#shell #bash #fondamentaux

## Déclaration

```bash
# Syntaxe recommandée
saluer() {
    echo "Bonjour, $1 !"
}

# Syntaxe alternative (équivalente, moins courante)
function saluer {
    echo "Bonjour, $1 !"
}

saluer "Alice"   # affiche : Bonjour, Alice !
```

## Paramètres d'une fonction

Une fonction reçoit ses arguments comme des paramètres positionnels (`$1`, `$2`...), exactement comme un script reçoit les siens.

```bash
afficher_info() {
    local nom="$1"
    local age="$2"
    echo "Nom : $nom, Âge : $age"
}

afficher_info "Alice" 30
```

## local : scope des variables

```bash
compteur=0

incrementer() {
    local compteur=0   # variable locale, indépendante de la globale
    compteur=$((compteur + 1))
    echo "Dans la fonction : $compteur"
}

incrementer
echo "Hors de la fonction : $compteur"   # toujours 0, inchangée
```

Sans `local`, une variable assignée dans une fonction est **globale par défaut** en Bash — contrairement à la plupart des langages où une fonction a un scope isolé par défaut.

## Valeur de retour : return vs echo

```bash
# return : code de sortie numérique (0-255), récupéré via $?
est_pair() {
    if (( $1 % 2 == 0 )); then
        return 0   # succès / vrai
    else
        return 1   # échec / faux
    fi
}

est_pair 4
echo $?   # 0

# echo : pour retourner une VALEUR (texte, nombre complexe), capturée via $()
calculer_carre() {
    echo $(( $1 * $1 ))
}

resultat=$(calculer_carre 5)
echo "$resultat"   # 25
```

`return` ne peut renvoyer qu'un entier entre 0 et 255 — c'est un code de statut (succès/échec), pas une valeur de calcul. Pour récupérer un résultat (chaîne, nombre quelconque), la fonction doit l'afficher avec `echo`, et l'appelant le capture via `$(...)`.

## Cas particuliers

> [!warning] Sans local, tout est global
> Oublier `local` dans une fonction qui modifie une variable du même nom qu'une variable globale écrase silencieusement cette dernière — source de bugs difficiles à tracer dans des scripts longs.

> [!tip] return pour la logique, echo pour la donnée
> Si la fonction sert à tester une condition (utilisée dans un `if fonction; then`), `return` suffit. Si elle doit produire une valeur utilisable ailleurs, `echo` + capture par `$()` est le pattern standard.

> [!info] Fonctions définies avant utilisation
> Une fonction doit être déclarée avant son premier appel dans le script — Bash lit et exécute séquentiellement, contrairement à des langages qui font du hoisting.
