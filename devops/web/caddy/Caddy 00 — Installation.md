#devops #caddy #fondamentaux #installation

## Ubuntu/Debian

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' \
  | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' \
  | sudo tee /etc/apt/sources.list.d/caddy-stable.list

sudo apt update && sudo apt install caddy
sudo systemctl status caddy   # le service systemd est configuré automatiquement
```

## RHEL/Rocky/Fedora

```bash
sudo dnf install 'dnf-command(copr)'
sudo dnf copr enable @caddy/caddy
sudo dnf install caddy
sudo systemctl enable --now caddy
```

## macOS

```bash
brew install caddy
```

## Docker

```bash
docker run -d -p 80:80 -p 443:443 \
  -v /path/to/Caddyfile:/etc/caddy/Caddyfile \
  -v caddy_data:/data \
  -v caddy_config:/config \
  caddy:latest
```

## Vérifier l'installation

```bash
caddy version
cat /etc/caddy/Caddyfile   # Caddyfile par défaut, créé par le paquet officiel
```

## Cas particuliers

> [!warning] Toujours installer la dernière version stable
> La branche 2.11 a corrigé plusieurs vulnérabilités en 2026, dont **CVE-2026-27586** (CVSS 9.1, corrigée en v2.11.1) : deux erreurs silencieusement ignorées dans `ClientAuthentication.provision()` faisaient échouer l'authentification par certificat client (mTLS) sans avertissement dès que le fichier CA était introuvable, corrompu ou mal configuré — le serveur acceptait alors n'importe quel certificat client signé par une autorité de confiance du système, sans que rien ne signale la défaillance. D'autres correctifs de sécurité ont suivi dans les versions patch ultérieures (2.11.2 à 2.11.4 au moins) — installer systématiquement la dernière version stable plutôt que de se fier à un numéro de version précis, et surveiller les advisories officiels.
> ```bash
> sudo apt update && sudo apt upgrade caddy
> caddy version
> ```

> [!info] Le Caddyfile par défaut se trouve dans `/etc/caddy/Caddyfile`
> Contrairement à Nginx et Apache (voir [[Nginx 00 — Installation]], [[Apache 00 — Installation]]), il n'y a qu'un seul emplacement de configuration standard, sans distinction Debian/RHEL pour son chemin.

## Pour aller plus loin

[[Caddy 01 — Qu'est-ce que Caddy]] pose les concepts, [[Caddy 02 — Le modèle mental du Caddyfile]] explique la structure avant le premier site.
