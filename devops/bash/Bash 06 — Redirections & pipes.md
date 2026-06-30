#shell #bash #redirections #pipes

## Les trois flux standards

| Flux | Descripteur | Usage |
|------|-------------|-------|
| `stdin` | `0` | Entrée standard (clavier, ou données reçues) |
| `stdout` | `1` | Sortie standard (résultat normal d'une commande) |
| `stderr` | `2` | Sortie d'erreur (messages d'erreur, séparés du résultat) |

## Redirections de sortie

```bash
commande > fichier.txt    # écrase fichier.txt avec stdout
commande >> fichier.txt    # ajoute stdout à la fin de fichier.txt
commande 2> erreurs.log    # redirige stderr uniquement
commande > out.log 2>&1     # redirige stdout ET stderr vers le même fichier
commande &> tout.log         # raccourci Bash équivalent à > fichier 2>&1
```

L'ordre compte dans `2>&1` : `2>&1` doit venir **après** la redirection de stdout pour fonctionner correctement.

```bash
# ❌ stderr part toujours vers le terminal (2>&1 dupliqué AVANT la redirection de stdout)
commande 2>&1 > fichier.txt

# ✅ stdout va dans le fichier, puis stderr suit stdout vers le même endroit
commande > fichier.txt 2>&1
```

## Redirection d'entrée

```bash
commande < fichier.txt    # fichier.txt devient l'entrée de la commande
sort < noms.txt              # trie le contenu de noms.txt
```

## Here-document

```bash
cat << EOF
Ligne 1
Ligne 2 avec $variable interprétée
EOF
```

```bash
# 'EOF' entre quotes : pas d'interprétation des variables/commandes à l'intérieur
cat << 'EOF'
Ceci affiche $variable littéralement, sans l'interpréter
EOF
```

Utile pour générer des fichiers de configuration multi-lignes directement depuis un script (ex. fichiers `.conf`, templates).

## Pipes

```bash
commande1 | commande2    # stdout de commande1 devient stdin de commande2

ps aux | grep nginx        # liste les processus, filtre ceux contenant "nginx"
cat fichier.log | grep ERROR | wc -l   # compte les lignes contenant "ERROR"
```

Un pipe ne transmet que `stdout` par défaut — `stderr` continue d'aller vers le terminal sauf redirection explicite.

## Cas particuliers

> [!warning] L'ordre de 2>&1 est crucial
> `2>&1 > fichier` ne fait pas ce qu'on croit : `2>&1` duplique stderr vers la destination actuelle de stdout (le terminal) **avant** que stdout soit redirigé vers le fichier. Toujours placer `2>&1` après la redirection de stdout.

> [!tip] /dev/null pour ignorer une sortie
> `commande > /dev/null 2>&1` exécute une commande en supprimant toute sortie (normale et erreur) — utile dans des scripts où seul le code de sortie (`$?`) compte.

> [!info] Pipe et sous-shells
> Chaque commande d'un pipe s'exécute dans son propre sous-shell — une variable modifiée dans la partie droite d'un pipe (`... | while read x; do var=$x; done`) ne persiste pas après le pipe, sauf usage de `lastpipe` (`shopt -s lastpipe`) ou de redirection alternative.
