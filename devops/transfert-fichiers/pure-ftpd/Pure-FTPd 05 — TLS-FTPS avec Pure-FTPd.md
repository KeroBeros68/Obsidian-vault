#devops #pure-ftpd #sécurité #avancé

## L'option TLS : trois niveaux, pas un simple on/off

```bash
echo "1" | sudo tee /etc/pure-ftpd/conf/TLS
```

| Valeur | Comportement |
|--------|--------------|
| `0` | TLS désactivé — FTP en clair uniquement |
| `1` | Accepte à la fois les connexions FTP classiques et FTPS — un client choisit |
| `2` | Refuse toute connexion qui n'active pas TLS |

> [!tip] `2` en production, `1` seulement en phase de transition
> `TLS 2` force tous les clients vers FTPS, cohérent avec le principe de ne jamais transmettre d'identifiants en clair (voir [[FTP 06 — Sécuriser FTP (FTPS)]]). `TLS 1` n'a de sens que pendant une migration progressive, le temps que tous les clients existants soient reconfigurés.

## Le certificat : un seul fichier combiné

Contrairement à des serveurs qui séparent certificat et clé privée dans deux fichiers distincts (voir par exemple [[MariaDB 12 — Galera Cluster, réplication synchrone multi-maître]] pour un cas de configuration TLS à deux fichiers), Pure-FTPd attend un unique fichier PEM combinant les deux :

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/pure-ftpd.pem \
  -out /etc/ssl/private/pure-ftpd.pem

sudo chmod 600 /etc/ssl/private/pure-ftpd.pem
sudo systemctl restart pure-ftpd
```

> [!warning] Le chemin du certificat est fixe, pas configurable par fichier d'option
> Pure-FTPd cherche son certificat à un emplacement conventionnel (`/etc/ssl/private/pure-ftpd.pem` sur la plupart des paquets Debian/Ubuntu) plutôt que via une directive `/etc/pure-ftpd/conf/` dédiée à un chemin personnalisé — vérifier la documentation du paquet de sa distribution en cas de doute sur l'emplacement exact attendu.

## Un certificat signé par une autorité reconnue, pour un usage public

Le certificat auto-signé généré ci-dessus suffit pour un usage interne ou un test, mais provoque un avertissement côté client pour un accès public — la même logique que pour n'importe quel service TLS exposé sur Internet (voir [[MySQL 24 — Chiffrement TLS]] pour la distinction générale entre chiffrement et vérification d'identité, transposable ici).

## Vérifier qu'une connexion utilise bien TLS

```bash
sudo journalctl -u pure-ftpd -n 50 | grep -i tls
```

```bash
# Côté client, forcer une connexion FTPS explicite
lftp -u alice, --tls-force ftp://serveur
```

## Pour aller plus loin

Une fois le transport chiffré, la maîtrise de l'usage disque et de la bande passante par utilisateur est couverte dans [[Pure-FTPd 06 — Quotas & limitation de bande passante]].

Sources : [Installation and Configuration of Pure-FTPd — Edoceo](https://edoceo.com/sys/pure-ftpd), [How To Configure PureFTPd To Accept TLS Sessions On Debian — HowtoForge](https://www.howtoforge.com/how-to-configure-pureftpd-to-accept-tls-sessions-on-debian-lenny)
