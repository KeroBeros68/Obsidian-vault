#devops #cron #pièges #erreurs #debugging

## 🪤 Piège 1 — PATH minimal

```bash
* * * * * script.sh          # ❌ introuvable, PATH cron ≠ PATH shell interactif
* * * * * /home/user/script.sh # ✅ chemin absolu
```

> [!warning] Environnement réduit
> Cron n'exécute pas `.bashrc`/`.profile`. Seules quelques variables minimales (`SHELL`, `PATH=/usr/bin:/bin`) sont définies par défaut.

---

## 🪤 Piège 2 — Le caractère `%` non échappé

```bash
0 3 * * * echo "sauvegarde du 100% du disque"   # ❌ interprété comme un saut de ligne
0 3 * * * echo "sauvegarde du 100\% du disque"  # ✅ échappé
```

> [!tip] Mémo
> `%` = retour à la ligne pour cron, sauf précédé d'un `\`.

---

## 🪤 Piège 3 — Sortie silencieuse ignorée

Par défaut, cron envoie stdout/stderr par mail local (MTA requis). Si aucun MTA n'est configuré, la sortie disparaît sans erreur visible.

> [!warning] Ce qui se passe concrètement
> Sans redirection explicite, un script qui échoue peut ne laisser aucune trace exploitable.

```bash
0 3 * * * /home/user/backup.sh >> /var/log/backup.log 2>&1  # ✅ traçabilité garantie
```

---

## 🪤 Piège 4 — `crontab -r` sans confirmation

```bash
crontab -r      # ❌ supprime toute la crontab, immédiatement, sans confirmation
crontab -i -r    # ✅ demande confirmation avant suppression
```

> [!warning] Ce qui se passe concrètement
> `crontab -r` est souvent tapée par réflexe après `crontab -e` (habitude `-e`/`-l`/`-r`) : aucune corbeille, aucun avertissement, la crontab entière disparaît.

---

## 🪤 Piège 5 — `/etc/cron.daily` qui ne s'exécute pas à l'heure attendue

Sur les distributions où anacron est installé (Debian par défaut), les répertoires `/etc/cron.{hourly,daily,weekly,monthly}` sont souvent pilotés par anacron plutôt que directement par `/etc/crontab`.

> [!warning] Ce qui se passe concrètement
> anacron ne garantit qu'un délai après le démarrage (champ `delay` en minutes), pas une heure fixe. Un job attendu à 6h25 peut s'exécuter à un autre moment, voire ne jamais sembler "programmé" à heure fixe.

> [!tip] Mémo
> Vérifier `systemctl status anacron` ou l'existence de `/etc/cron.d/anacron` avant de chercher pourquoi un job de `cron.daily` ne s'est pas déclenché à l'heure prévue.

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Commande introuvable | Toujours utiliser des chemins absolus |
| `%` interprété comme newline | Échapper avec `\%` |
| Sortie perdue silencieusement | Rediriger vers un fichier de log (`>> fichier 2>&1`) |
| Suppression accidentelle de la crontab | Utiliser `crontab -i -r` |
| `cron.daily` à une heure inattendue | Vérifier si anacron pilote `run-parts` à sa place |
