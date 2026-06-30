#shell #bash #robustesse #avancé

## Le problème : Bash continue après une erreur, par défaut

Par défaut, si une commande échoue dans un script Bash, l'exécution **continue** avec la ligne suivante — contrairement à la plupart des langages, où une erreur non gérée arrête le programme.

```bash
#!/usr/bin/env bash
cd /chemin/inexistant    # échoue, mais le script continue !
rm -rf *                  # s'exécute dans le MAUVAIS dossier
```

## set -euo pipefail

```bash
#!/usr/bin/env bash
set -euo pipefail
```

| Option | Effet |
|--------|-------|
| `-e` | Arrête le script immédiatement si une commande échoue (code de sortie non nul) |
| `-u` | Arrête le script si une variable non définie est utilisée (typo de nom de variable) |
| `-o pipefail` | Un pipe (`cmd1 \| cmd2`) échoue si **n'importe quelle** commande de la chaîne échoue, pas seulement la dernière |

```bash
set -u
firstName="Aaron"
fullName="$firstname Maxwell"   # ❌ typo : firstname au lieu de firstName
echo "$fullName"
# Avec -u : erreur explicite "firstname: unbound variable", arrêt immédiat
# Sans -u : silencieusement vide, bug difficile à repérer
```

```bash
set -o pipefail
grep "erreur" /fichier/inexistant.log | sort
echo $?
# Sans pipefail : $? reflète le code de sortie de 'sort' (souvent 0, donc "succès")
# Avec pipefail : $? reflète l'échec de 'grep', le pipe entier est considéré en échec
```

## Pourquoi pipefail change tout

Sans `pipefail`, le code de sortie d'un pipe est celui de sa **dernière** commande uniquement. Une commande de tri qui réussit sur un flux vide masque silencieusement l'échec de la commande qui l'alimente — exactement le genre de bug qui passe inaperçu en CI jusqu'à un incident en production.

## trap : exécuter du code au moment du nettoyage

```bash
#!/usr/bin/env bash
set -euo pipefail

fichier_temp=$(mktemp)

nettoyer() {
    rm -f "$fichier_temp"
    echo "Nettoyage effectué"
}
trap nettoyer EXIT   # 'nettoyer' s'exécute quoi qu'il arrive : succès, erreur, ou Ctrl+C

echo "data" > "$fichier_temp"
# ... traitement ...
```

`trap ... EXIT` garantit que le nettoyage a lieu même si le script s'arrête en plein milieu à cause de `set -e` — utile pour supprimer des fichiers temporaires ou libérer des verrous.

## Cas particuliers

> [!warning] set -e n'attrape pas tout
> `set -e` ne se déclenche pas dans certains contextes : une commande utilisée dans une condition (`if commande; then`), une commande suivie de `||`, ou — piège classique — `((variable++))` quand la variable vaut `0` (son code de sortie reflète la valeur arithmétique, pas un succès/échec), ce qui peut interrompre le script de façon inattendue.

> [!tip] || true pour neutraliser une erreur volontaire
> Quand un échec ponctuel est acceptable et ne doit pas interrompre le script malgré `set -e`, ajouter `|| true` après la commande : `commande_qui_peut_echouer || true`.

> [!info] inherit_errexit pour les sous-shells
> Par défaut, `set -e` ne se propage pas automatiquement dans une substitution de commande (`$(...)`). `shopt -s inherit_errexit` (Bash ≥ 4.4) corrige ce comportement pour un script encore plus prévisible.
