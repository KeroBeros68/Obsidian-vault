#devops #cron #environnement

## Environnement minimal des jobs

Cron n'exécute ni `.bashrc` ni `.profile`. Chaque job démarre avec un environnement réduit à quelques variables par défaut : `SHELL=/bin/sh`, `PATH=/usr/bin:/bin`, ainsi que `HOME` et `LOGNAME` déduits du propriétaire de la crontab.

Ces variables peuvent être redéfinies en tête de crontab, avant les lignes de planification :

```bash
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
MAILTO=""

0 3 * * * /home/user/backup.sh
```

## Cas courants

| Situation | Syntaxe / Approche |
|---|---|
| Recevoir les erreurs par mail | `MAILTO=admin@example.com` |
| Désactiver l'envoi de mail | `MAILTO=""` |
| Changer le shell d'exécution | `SHELL=/bin/bash` |
| Élargir le PATH minimal | `PATH=/usr/local/bin:/usr/bin:/bin` |

> [!warning] `MAILTO=""` n'est pas la même chose qu'une variable absente
> Sans `MAILTO`, cron envoie la sortie au propriétaire local du job (mail requiert un MTA fonctionnel). Avec `MAILTO=""` explicitement vide, l'envoi est désactivé — la sortie non redirigée est alors purement perdue.

## anacron — rattraper les tâches manquées

anacron complète cron pour les machines qui ne tournent pas en continu (portables, serveurs éteints la nuit). Il ne planifie pas à une heure précise, mais garantit qu'une tâche s'exécute au moins une fois par période donnée, avec un délai après le démarrage.

```
# /etc/anacrontab — period(jours) delay(minutes) identifiant commande
1   5   cron.daily    run-parts /etc/cron.daily
7   10  cron.weekly   run-parts /etc/cron.weekly
30  15  cron.monthly  run-parts /etc/cron.monthly
```

> [!info] `run-parts`
> Exécute chaque script exécutable d'un répertoire, dans l'ordre alphabétique. C'est le mécanisme qui relie `/etc/cron.{daily,weekly,monthly}/` à `/etc/crontab` (cron seul) ou à `/etc/anacrontab` (anacron).

> [!warning] anacron ne respecte pas une heure fixe
> Contrairement à une ligne crontab classique, anacron ne garantit qu'un délai (en minutes) après le démarrage du système ou du service — pas un horaire précis. Un job `cron.daily` peut donc s'exécuter à des heures différentes selon quand la machine était allumée.

> [!tip] Identifier qui exécute `/etc/cron.daily`
> `systemctl status anacron` (ou l'existence de `/etc/cron.d/anacron`) indique si anacron a pris le relais de `/etc/crontab` pour les répertoires `cron.{hourly,daily,weekly,monthly}`. Sur une distribution sans anacron installé, c'est `/etc/crontab` qui déclenche directement `run-parts` à heure fixe.
