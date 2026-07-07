#devops #cron #services

## Identifier le service selon la distribution

| Distribution | Paquet | Nom du service systemd |
|---|---|---|
| Debian / Ubuntu | `cron` (préinstallé) | `cron` |
| RHEL / CentOS / Fedora / Rocky | `cronie` (préinstallé) | `crond` |
| Arch Linux | `cronie` (à installer manuellement) | `cronie` |

> [!info] Arch Linux ne préinstalle pas de daemon cron
> Cohérent avec sa philosophie minimaliste : `cronie` doit être installé explicitement.

```bash
# Arch Linux — installation
sudo pacman -S cronie   # ✅ paquet officiel recommandé, fournit crond + crontab + anacron
```

## Démarrer et activer le service (systemd)

```bash
# Debian/Ubuntu
sudo systemctl status cron
sudo systemctl enable --now cron

# RHEL/CentOS/Fedora
sudo systemctl status crond
sudo systemctl enable --now crond

# Arch Linux
sudo systemctl status cronie
sudo systemctl enable --now cronie    # ✅ démarre + active au boot en une commande
```

## Vérifier l'exécution effective des jobs

```bash
# Debian/Ubuntu — logué dans syslog
grep CRON /var/log/syslog

# RHEL/CentOS/Fedora — fichier dédié
sudo tail -f /var/log/cron

# Arch Linux — pas de fichier dédié par défaut, passer par journalctl
journalctl -u cronie -f
```

> [!warning] `systemctl status` confirme le daemon, pas les jobs
> Un statut `active (running)` signifie que `crond` tourne, pas qu'un job précis s'est exécuté correctement. Toujours croiser avec les logs ou une redirection explicite de sortie dans le job lui-même.

> [!tip] Désinstaller/désactiver proprement (Arch)
> ```bash
> sudo systemctl stop cronie.service
> sudo systemctl disable cronie.service
> ```
> Nécessaire avant toute désinstallation du paquet, sous peine de laisser un service fantôme référencé par systemd.
