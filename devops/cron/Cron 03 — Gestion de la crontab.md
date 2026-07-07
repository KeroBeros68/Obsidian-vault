#devops #cron #crontab

## Emplacements des crontabs

Cron lit trois familles de sources, chacune avec un format et un usage différents.

| Source | Format | Édition |
|---|---|---|
| Spool utilisateur (`/var/spool/cron/crontabs/<user>`) | 5 champs, pas de champ utilisateur | `crontab -e` uniquement |
| `/etc/crontab` | 6 champs, champ utilisateur obligatoire | édition directe (root) |
| `/etc/cron.d/*` | 6 champs, champ utilisateur obligatoire | édition directe (root), utilisé par les paquets pour installer leurs propres jobs |
| `/etc/cron.{hourly,daily,weekly,monthly}/` | scripts exécutables, pas de syntaxe cron | dépôt de scripts, déclenchés via `run-parts` |

## Commandes de gestion

| Commande | Effet |
|---|---|
| `crontab -e` | Éditer sa propre crontab (créée si absente) |
| `crontab -l` | Lister le contenu de sa crontab |
| `crontab -r` | Supprimer sa crontab entière |
| `crontab -i -r` | Supprimer avec confirmation préalable |
| `crontab -u <user> -e` | Éditer la crontab d'un autre utilisateur (droits root requis) |

## Comportement ou subtilité

```bash
EDITOR=vim crontab -e     # ✅ choisit l'éditeur (sinon `vi` par défaut sur la plupart des distros)
crontab -r                 # ❌ supprime immédiatement, sans confirmation ni corbeille
crontab -i -r               # ✅ demande confirmation avant suppression
```

> [!warning] `cron.allow` / `cron.deny`
> Si `/etc/cron.allow` existe, seuls les utilisateurs qui y figurent peuvent utiliser `crontab`. Sinon, `/etc/cron.deny` bloque les utilisateurs listés. Sans aucun des deux fichiers, le comportement par défaut dépend de la distribution (généralement tout le monde autorisé).

> [!tip] Ne jamais éditer le spool à la main
> Modifier directement un fichier dans `/var/spool/cron/crontabs/` contourne le verrou et la validation de syntaxe de `crontab -e` — la modification peut être ignorée ou corrompre le fichier. Toujours passer par la commande `crontab`.
