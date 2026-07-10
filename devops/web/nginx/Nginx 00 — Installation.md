#devops #nginx #fondamentaux #installation

## Ubuntu/Debian

```bash
# Version des dépôts (simple, parfois en retard sur la dernière version)
sudo apt update && sudo apt install nginx
sudo systemctl enable --now nginx
```

```bash
# Dépôt officiel Nginx (version récente)
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
  | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
  http://nginx.org/packages/ubuntu $(lsb_release -cs) nginx" \
  | sudo tee /etc/apt/sources.list.d/nginx.list

sudo apt update && sudo apt install nginx
sudo systemctl enable --now nginx
```

## RHEL/Rocky/Fedora

```bash
sudo tee /etc/yum.repos.d/nginx.repo <<'EOF'
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/rhel/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
EOF

sudo dnf install nginx
sudo systemctl enable --now nginx
```

## Vérifier l'installation

```bash
systemctl status nginx   # doit afficher "active (running)"
nginx -v                 # version installée
curl -I http://localhost # doit retourner HTTP/1.1 200 OK
```

Ouvrir `http://votre-ip` dans un navigateur doit afficher la page « Welcome to nginx ».

## Cas particuliers

> [!info] Debian/Ubuntu vs RHEL : deux organisations de fichiers
> Debian/Ubuntu ajoutent en général `sites-available/` et `sites-enabled/` (liens symboliques activant un site), en plus de `conf.d/`. RHEL/Rocky place tout directement dans `conf.d/`. Les deux fonctionnent, voir [[Nginx 04 — Structure du fichier de configuration]] pour la hiérarchie des contextes elle-même, indépendante de cette convention de fichiers.

> [!warning] `docker.io`/paquet distro vs dépôt officiel
> Le paquet Nginx des dépôts par défaut d'une distribution est parfois en retard de plusieurs versions mineures sur le dépôt officiel `nginx.org` — à privilégier si des fonctionnalités récentes (HTTP/3, directives ajoutées dans une version récente) sont nécessaires.

## Pour aller plus loin

Une fois installé, [[Nginx 01 — Qu'est-ce que Nginx]] pose les concepts, et [[Nginx 05 — Server blocks & virtual hosting]] sert le premier site.
