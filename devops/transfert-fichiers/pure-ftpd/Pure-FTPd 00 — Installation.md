#devops #pure-ftpd #installation #fondamentaux

## Installer sur Debian/Ubuntu

```bash
sudo apt update
sudo apt install pure-ftpd
sudo systemctl status pure-ftpd
```

```bash
ftp localhost
-- Connected to localhost
-- 220 ---------- Welcome to Pure-FTPd ----------
```

> [!info] Deux paquets, un rôle chacun
> Sur Debian/Ubuntu, `pure-ftpd` installe le démon et ses utilitaires, tandis que `pure-ftpd-common` porte le mécanisme de configuration propre à la distribution (voir [[Pure-FTPd 01 — Le mécanisme de configuration (wrapper Debian)]]) — installé automatiquement comme dépendance.

## Positionnement par rapport au module FTP générique

Pure-FTPd implémente le protocole FTP couvert dans le module de base (voir [[FTP — Index des fiches]]) : canaux de contrôle/données, modes actif/passif, FTPS. Ce qui suit se concentre sur ce qui est **spécifique à Pure-FTPd** — en particulier son mécanisme de configuration, assez différent des autres serveurs FTP.

> [!warning] Pure-FTPd n'a pas de fichier de configuration monolithique classique
> Contrairement à un `nginx.conf` ou un `vsftpd.conf` unique, Pure-FTPd se configure nativement via des **arguments en ligne de commande** passés au démon. Sur Debian/Ubuntu, un mécanisme dédié traduit des fichiers de configuration en ces arguments — détaillé dans [[Pure-FTPd 01 — Le mécanisme de configuration (wrapper Debian)]], à lire avant toute tentative de configuration.

## Vérifier le service

```bash
sudo systemctl restart pure-ftpd
sudo journalctl -u pure-ftpd -n 50
```

## Pour aller plus loin

Le mécanisme de configuration, particularité la plus structurante de Pure-FTPd, est détaillé dans [[Pure-FTPd 01 — Le mécanisme de configuration (wrapper Debian)]].

Sources : [Pure-FTPd Documentation — pureftpd.org](https://www.pureftpd.org/project/pure-ftpd/doc/), [Debian package pure-ftpd-common — packages.debian.org](https://packages.debian.org/bookworm/pure-ftpd-common)
