#devops #apache #fondamentaux #installation

## Debian/Ubuntu vs RHEL/Rocky : deux noms, deux organisations

Apache s'appelle `apache2` sur Debian/Ubuntu et `httpd` sur RHEL/Rocky/Fedora — deux noms de paquet, de service et d'arborescence de configuration différents pour le même logiciel.

```bash
# Identifier sa distribution avant de choisir les commandes
cat /etc/os-release | grep -E '^(NAME|ID)='
```

## Ubuntu/Debian

```bash
sudo apt update && sudo apt install apache2
sudo systemctl enable --now apache2
sudo ufw allow 'Apache Full'   # ouvre 80 et 443
```

## RHEL/Rocky/Fedora

```bash
sudo dnf install httpd
sudo systemctl enable --now httpd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

## Vérifier l'installation

```bash
systemctl status apache2   # ou httpd — doit afficher "active (running)"
curl -I http://localhost   # doit retourner HTTP/1.1 200 OK
```

Ouvrir `http://votre-ip` dans un navigateur doit afficher la page par défaut d'Apache.

## Différences à retenir dès maintenant

| Aspect | Debian/Ubuntu | RHEL/Rocky |
|--------|-------------------|----------------|
| Paquet | `apache2` | `httpd` |
| Service | `apache2` | `httpd` |
| Config principale | `/etc/apache2/apache2.conf` | `/etc/httpd/conf/httpd.conf` |
| Utilisateur d'exécution | `www-data` | `apache` |
| Pare-feu | `ufw` | `firewalld` |

Détail complet de l'arborescence de configuration dans [[Apache 04 — Structure du fichier de configuration]].

## Cas particuliers

> [!warning] Ne pas mélanger les commandes des deux mondes
> `a2ensite`/`a2enmod` n'existent que sur Debian/Ubuntu (scripts fournis par le paquet, pas par Apache lui-même) — sur RHEL/Rocky, activer un site ou un module se fait en éditant directement les fichiers de configuration. Voir [[Apache 03 — Modules (a2enmod)]] et [[Apache 04 — Structure du fichier de configuration]].
