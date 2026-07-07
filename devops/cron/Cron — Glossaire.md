#devops #cron #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **crontab** | Fichier de configuration listant les tâches planifiées d'un utilisateur. Se manipule via `crontab -e` (édition), `crontab -l` (liste), `crontab -r` (suppression). |
| **cron (daemon)** | Processus système en arrière-plan qui lit les crontabs à intervalle fixe (généralement chaque minute) et lance les commandes dues. |
| **spool** | Répertoire où sont stockées les crontabs individuelles des utilisateurs, typiquement `/var/spool/cron/crontabs/`. |
| **MAILTO** | Variable d'environnement définissable en tête de crontab pour rediriger la sortie des jobs vers une adresse mail au lieu de l'utilisateur local. |
| **@reboot** | Alias spécial exécutant la commande une seule fois, au démarrage du système. |
| **anacron** | Complément à cron pour les machines qui ne tournent pas en continu (portables, serveurs éteints la nuit) : rattrape les tâches manquées au prochain démarrage. |
| **crond** | Nom du binaire/service cron sur les distributions RHEL/CentOS/Fedora (équivalent de `cron` sous Debian/Ubuntu, `cronie` sous Arch Linux). |
| **cron.allow / cron.deny** | Fichiers de contrôle d'accès à `crontab` : liste blanche (`cron.allow`) ou liste noire (`cron.deny`) des utilisateurs autorisés à planifier des tâches. |
| **run-parts** | Utilitaire qui exécute tous les scripts exécutables d'un répertoire dans l'ordre alphabétique ; relie `/etc/cron.{hourly,daily,weekly,monthly}/` à `/etc/crontab` ou à `/etc/anacrontab`. |
| **anacrontab** | Fichier de configuration d'anacron (`/etc/anacrontab`), avec un format différent de la crontab classique : période en jours, délai en minutes, identifiant, commande. |
