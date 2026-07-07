#devops #cron #bases

## Anatomie d'une ligne crontab

Une ligne crontab associe un moment d'exécution à une commande, via cinq champs temporels séparés par des espaces.

```bash
# ┌───────────── minute (0–59)
# │ ┌───────────── heure (0–23)
# │ │ ┌───────────── jour du mois (1–31)
# │ │ │ ┌───────────── mois (1–12)
# │ │ │ │ ┌───────────── jour de la semaine (0–7, 0 et 7 = dimanche)
# │ │ │ │ │
  * * * * *  commande_a_executer
```

## Cas courants

| Situation                     | Syntaxe / Approche |
| ----------------------------- | ------------------ |
| Toutes les minutes            | `* * * * *`        |
| Tous les jours à 2h00         | `0 2 * * *`        |
| Toutes les 15 minutes         | `*/15 * * * *`     |
| Chaque lundi à 9h00           | `0 9 * * 1`        |
| Le 1er de chaque mois         | `0 0 1 * *`        |
| Au démarrage du système       | `@reboot commande` |
| Une fois par jour (raccourci) | `@daily commande`  |

> [!info] Alias spéciaux disponibles
> `@reboot`, `@yearly`/`@annually`, `@monthly`, `@weekly`, `@daily`/`@midnight`, `@hourly` remplacent les 5 champs temporels dans les implémentations courantes (cronie, Vixie cron).

## Comportement ou subtilité

```bash
# Crontab utilisateur (via `crontab -e`) : 5 champs, pas de champ utilisateur
0 3 * * * /home/user/backup.sh          # ✅ chemin absolu, s'exécute avec l'utilisateur propriétaire de la crontab

# Fichier système (/etc/crontab, /etc/cron.d/*) : 6 champs, avec utilisateur
0 3 * * * root /usr/local/bin/backup.sh  # ✅ champ utilisateur obligatoire ici

# Erreur silencieuse classique
* * * * * script.sh                      # ❌ PATH minimal, "script.sh" introuvable si non dans le PATH cron

* * * * * echo "100% termine" >> log.txt # ❌ le "%" est interprété comme un saut de ligne par cron
* * * * * echo "100\%termine" >> log.txt # ✅ "%" échappé avec un backslash
```

> [!warning] Le `%` n'est pas un caractère littéral en cron
> Dans la commande, `%` est traité comme un retour à la ligne : tout ce qui suit devient le contenu de l'entrée standard (stdin) de la commande, sauf s'il est échappé par `\%`.

> [!tip] Toujours utiliser des chemins absolus
> Cron n'hérite pas de l'environnement du shell interactif (`.bashrc`, `PATH` personnalisé). Préciser le chemin complet des exécutables et des scripts évite la majorité des échecs silencieux.
