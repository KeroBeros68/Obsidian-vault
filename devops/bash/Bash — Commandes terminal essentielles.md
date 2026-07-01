#shell #bash #commandes #référence #admin-sys

## Fichiers & dossiers

| Commande | Utilité | Exemple |
|----------|---------|---------|
| `ls` | Lister le contenu d'un dossier | `ls -la` *(détails complets, fichiers cachés inclus)* |
| `cd` | Changer de dossier courant | `cd /var/log` |
| `pwd` | Afficher le chemin du dossier courant | `pwd` |
| `cp` | Copier un fichier ou dossier | `cp -r source/ destination/` *(`-r` pour un dossier)* |
| `mv` | Déplacer ou renommer | `mv ancien.txt nouveau.txt` |
| `rm` | Supprimer | `rm -rf dossier/` *(`-rf` : récursif, sans confirmation — danger)* |
| `mkdir` | Créer un dossier | `mkdir -p projet/src/utils` *(`-p` crée les parents manquants)* |
| `find` | Rechercher des fichiers selon des critères | `find /var/log -name "*.log" -mtime -7` *(modifiés < 7 jours)* |
| `du` | Afficher l'espace disque utilisé | `du -sh dossier/` *(`-s` résumé, `-h` lisible)* |
| `df` | Afficher l'espace disque disponible par partition | `df -h` |
| `tar` | Archiver / compresser | `tar -czvf archive.tar.gz dossier/` *(création, gzip, verbeux)* |
| `grep` | Rechercher du texte dans des fichiers | `grep -r "ERROR" /var/log/` *(`-r` récursif)* |

## Processus

| Commande | Utilité | Exemple |
|----------|---------|---------|
| `ps` | Lister les processus en cours | `ps aux` *(tous les processus, tous utilisateurs)* |
| `top` / `htop` | Vue temps réel des processus et ressources | `htop` *(version améliorée, interactive)* |
| `kill` | Envoyer un signal à un processus (par PID) | `kill -9 1234` *(`-9` : forcer l'arrêt, SIGKILL)* |
| `killall` | Tuer tous les processus portant un nom donné | `killall nginx` |
| `jobs` | Lister les tâches en arrière-plan du shell courant | `jobs -l` |
| `bg` / `fg` | Passer une tâche en arrière-plan / premier plan | `fg %1` *(rappelle la tâche n°1)* |
| `nohup` | Lancer une commande insensible à la fermeture du terminal | `nohup ./script.sh &` |
| `systemctl` | Gérer les services systemd | `systemctl status nginx` |

```bash
# Trouver puis tuer un processus par nom
ps aux | grep nginx
kill -9 <PID>
```

## Réseau

| Commande | Utilité                                            | Exemple                                                    |
| -------- | -------------------------------------------------- | ---------------------------------------------------------- |
| `curl`   | Effectuer une requête HTTP / télécharger           | `curl -I https://example.com` *(`-I` : headers seulement)* |
| `wget`   | Télécharger un fichier                             | `wget https://example.com/fichier.zip`                     |
| `ssh`    | Se connecter à une machine distante                | `ssh user@serveur.example.com`                             |
| `scp`    | Copier un fichier vers/depuis une machine distante | `scp fichier.txt user@serveur:/chemin/`                    |
| `ping`   | Tester la connectivité réseau                      | `ping -c 4 example.com` *(`-c 4` : 4 paquets puis arrêt)*  |
| `ss`     | Lister les connexions/sockets réseau               | `ss -tulnp` *(TCP/UDP, écoute, numérique, process)*        |
| `ip`     | Gérer interfaces, adresses, routes                 | `ip addr show` *(affiche les IP des interfaces)*           |
| `dig`    | Interroger le DNS                                  | `dig example.com +short`                                   |

```bash
# Vérifier quel processus écoute sur le port 80
sudo ss -tulnp | grep :80
```

## Permissions & droits

| Commande | Utilité | Exemple |
|----------|---------|---------|
| `chmod` | Modifier les permissions d'un fichier | `chmod 755 script.sh` *(rwx pour le propriétaire, rx pour les autres)* |
| `chown` | Changer le propriétaire d'un fichier | `chown alice:devs fichier.txt` |
| `sudo` | Exécuter une commande avec privilèges élevés | `sudo systemctl restart nginx` |
| `su` | Changer d'utilisateur dans la session courante | `su - alice` |
| `id` | Afficher l'UID/GID de l'utilisateur courant | `id` |
| `whoami` | Afficher l'utilisateur courant | `whoami` |

```bash
# Lecture/écriture/exécution pour le propriétaire, lecture/exécution pour les autres
chmod 755 deploy.sh
```

## Cas particuliers

> [!warning] netstat est déprécié
> `netstat` était l'outil historique pour inspecter les connexions réseau, mais il est officiellement déprécié et de moins en moins présent par défaut sur les distributions récentes. `ss` (du paquet `iproute2`) le remplace : plus rapide, plus précis, activement maintenu.

> [!tip] rm -rf, toujours vérifier avant
> Avant d'exécuter un `rm -rf` avec une variable (`rm -rf "$dossier"`), tester d'abord avec `echo "$dossier"` ou `ls "$dossier"` pour confirmer la cible — une variable vide ou mal quotée transforme `rm -rf "$dossier"/` en suppression catastrophique.

> [!info] aux vs -ef pour ps
> `ps aux` (style BSD) et `ps -ef` (style POSIX) affichent des informations similaires avec un format légèrement différent — les deux sont largement utilisés, le choix est surtout une question d'habitude.
