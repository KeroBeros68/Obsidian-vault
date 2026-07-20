#bdd #mysql #fondamentaux #installation

## MySQL vs MariaDB : ne pas les confondre au moment d'installer

Beaucoup de distributions (Debian, Rocky Linux, Fedora) embarquent **MariaDB** dans leurs dépôts par défaut, pas MySQL — un fork créé en 2009 par le fondateur original de MySQL après le rachat par Oracle.

| | MySQL (Oracle) | MariaDB |
|---|--------------------|-------------|
| Éditeur | Oracle Corporation | MariaDB Foundation / MariaDB plc |
| Version LTS actuelle | 8.4 (support 8 ans) | 11.4 (support 5 ans) |
| Réplication | GTID, Group Replication, InnoDB Cluster | GTID (format différent), Galera Cluster |
| Plugin auth par défaut | `caching_sha2_password` | `mysql_native_password` |
| Performance Schema | Complet | Partiel |

Les deux partagent le protocole réseau et l'essentiel de la syntaxe SQL, mais **ne sont pas interchangeables** en production — leurs extensions spécifiques (Group Replication vs Galera, par exemple) ne sont pas compatibles entre elles.

> [!warning] Ne jamais installer MySQL et MariaDB sur la même machine
> Les paquets entrent en conflit (mêmes noms de fichiers, même port `3306`, même socket). Si MariaDB est déjà présent, le désinstaller complètement avant d'installer MySQL (voir Dépannage plus bas).

Pour obtenir MySQL 8.4 LTS (voir [[MySQL 09 — Versions (LTS vs Innovation) & arborescence des fichiers]]) plutôt que la version figée ou MariaDB fournie par une distribution, le **dépôt officiel Oracle** est le chemin le plus fiable.

## Installer depuis le dépôt officiel Oracle (Debian/Ubuntu)

```bash
# 1. Télécharger le paquet de configuration du dépôt (le numéro de version change régulièrement, vérifier la page de téléchargement officielle)
cd /tmp
wget https://dev.mysql.com/get/mysql-apt-config_0.8.36-1_all.deb

# 2. Installer ce paquet (non-interactif : sélectionne directement MySQL 8.4 LTS)
sudo DEBIAN_FRONTEND=noninteractive dpkg -i mysql-apt-config_0.8.36-1_all.deb

# 3. Mettre à jour les sources et installer
sudo apt update
sudo apt install -y mysql-server
```

> [!info] Sans `DEBIAN_FRONTEND=noninteractive`
> Un écran `dialog` propose de choisir entre MySQL 8.4 LTS et les versions Innovation — LTS est déjà présélectionné, valider avec `OK` suffit dans l'immense majorité des cas.

## Installer depuis le dépôt officiel Oracle (RHEL/Rocky)

```bash
# 1. Installer le dépôt YUM officiel (le numéro de version change régulièrement)
sudo dnf install -y https://dev.mysql.com/get/mysql84-community-release-el9-3.noarch.rpm

# 2. Installer MySQL
sudo dnf install -y mysql-community-server

# 3. Démarrer et activer le service — RHEL ne le fait PAS automatiquement, contrairement à Debian
sudo systemctl enable --now mysqld
```

> [!warning] RHEL/Rocky 8 : désactiver le module MySQL de la distribution d'abord
> `sudo dnf -qy module disable mysql` avant l'installation — étape inutile sur EL9.

## La différence fondamentale Debian vs RHEL : le mot de passe root

| | Debian/Ubuntu | RHEL/Rocky |
|---|-------------------|----------------|
| Mot de passe root | Proposé à l'installation (peut être laissé vide → authentification par socket Unix) | Mot de passe **temporaire** généré automatiquement |
| Où le trouver | — | `sudo grep 'temporary password' /var/log/mysqld.log` |
| Action requise | Optionnelle | **Obligatoire** : `ALTER USER 'root'@'localhost' IDENTIFIED BY '...'` dès la première connexion |

> [!warning] Source fréquente de « pourquoi je ne peux pas me connecter »
> Sur RHEL/Rocky, ne pas récupérer et changer le mot de passe temporaire avant la première tentative de connexion est la cause la plus courante d'un blocage post-installation.

```bash
# Automatiser le mot de passe root sur Debian/Ubuntu (installation non interactive)
echo "mysql-community-server mysql-community-server/root-pass password VotreMotDePasse" | sudo debconf-set-selections
echo "mysql-community-server mysql-community-server/re-root-pass password VotreMotDePasse" | sudo debconf-set-selections
```

## Vérifier l'installation

Le nom du service diffère aussi selon la distribution :

```bash
# Debian/Ubuntu
sudo systemctl status mysql
sudo systemctl {stop,start,restart,enable} mysql

# RHEL/Rocky
sudo systemctl status mysqld
sudo systemctl {stop,start,enable} mysqld
```

```bash
mysql -u root -p
```

```sql
SELECT VERSION();
\s   -- affiche notamment "Connection: Localhost via UNIX socket"
```

> [!info] MySQL n'a pas de vrai `reload`
> Contrairement à PostgreSQL, la plupart des paramètres InnoDB ne se modifient qu'avec un `restart` ou dynamiquement en SQL (`SET GLOBAL`/`SET PERSIST`) — pas de `systemctl reload` équivalent. Détaillé dans un futur guide de configuration.

## Emplacements de configuration selon la distribution

| Distribution | Configuration serveur | Datadir |
|-------------------|----------------------------|-------------|
| Debian/Ubuntu | `/etc/mysql/mysql.conf.d/mysqld.cnf` | `/var/lib/mysql/` |
| RHEL/Rocky | `/etc/my.cnf` | `/var/lib/mysql/` |

Sur Debian, `/etc/mysql/my.cnf` inclut `conf.d/` (options client) et `mysql.conf.d/` (configuration serveur) — voir [[MySQL 09 — Versions (LTS vs Innovation) & arborescence des fichiers]] pour l'arborescence complète du datadir.

## Vérifier les paramètres clés après installation

```sql
SELECT @@innodb_buffer_pool_size/1024/1024 AS buffer_pool_mb,
       @@max_connections, @@bind_address, @@log_bin, @@gtid_mode;
```

| Paramètre | Valeur par défaut | Signification |
|-----------|------------------------|--------------------|
| `innodb_buffer_pool_size` | 128 Mo | Beaucoup trop petit pour la production — voir [[MySQL 05 — InnoDB — le buffer pool]] |
| `max_connections` | 151 | Connexions simultanées maximum |
| `bind_address` | `127.0.0.1` | Aucun accès réseau distant par défaut |
| `log_bin` | ON (MySQL 8.4) | Binary log activé par défaut depuis la 8.4 — voir [[MySQL 07 — Binary log — réplication & PITR]] |
| `gtid_mode` | OFF | À activer explicitement pour la réplication |

## Sécuriser l'installation initiale

```bash
sudo mysql_secure_installation
```

| Question | Réponse recommandée | Pourquoi |
|----------|--------------------------|----------|
| Setup VALIDATE PASSWORD component? | Yes (si proposé) | Politique de complexité des mots de passe |
| Change root password? | No (si déjà changé) | — |
| Remove anonymous users? | Yes | Supprime les comptes sans nom, connexion sans authentification |
| Disallow root login remotely? | Yes | Empêche `root` depuis un autre serveur |
| Remove test database? | Yes | Supprime la base `test`, accessible à tous |
| Reload privilege tables? | Yes | Applique les changements immédiatement |

## Créer un premier utilisateur d'administration

```sql
CREATE USER 'labadmin'@'localhost' IDENTIFIED BY 'MotDePasseFort!2026';
GRANT ALL PRIVILEGES ON *.* TO 'labadmin'@'localhost' WITH GRANT OPTION;
```

> [!warning] `GRANT ALL` n'est pas pour la production
> Cette syntaxe donne tous les droits sur toutes les bases — acceptable pour un lab ou un compte d'administration ponctuel, mais en production les comptes doivent recevoir uniquement les permissions minimales nécessaires.

> [!info] `FLUSH PRIVILEGES` est inutile après `CREATE USER`/`GRANT`/`REVOKE`
> Ces instructions prennent effet immédiatement — `FLUSH PRIVILEGES` ne sert que lorsqu'une modification directe des tables `mysql.user`/`mysql.db` a été faite (déconseillé, voir [[MySQL — Pièges classiques]]).

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|--------------------|----------|
| `Bind on TCP/IP port: Address already in use` | Un autre processus occupe déjà le port 3306 | `ss -tlnp \| grep 3306` |
| `--initialize specified but the data directory has files in it` | Le datadir contient déjà des fichiers | Vider `/var/lib/mysql/*` ou utiliser un autre datadir |
| `InnoDB: Unable to lock ./ibdata1` | Une autre instance `mysqld` tourne déjà | `ps aux \| grep mysqld`, arrêter l'instance en conflit |
| Service introuvable | Mauvais nom de service | `mysql` (Debian) vs `mysqld` (RHEL) |

```bash
# Logs d'erreur
sudo cat /var/log/mysql/error.log | tail -30    # Debian
sudo cat /var/log/mysqld.log | tail -30          # RHEL
sudo journalctl -u mysql --no-pager -n 30        # Debian
sudo journalctl -u mysqld --no-pager -n 30       # RHEL
```

### Mot de passe root perdu

```bash
sudo systemctl stop mysql
sudo mysqld --skip-grant-tables --skip-networking --user=mysql &
mysql -u root
```

```sql
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NouveauMotDePasse';
```

```bash
sudo kill $(cat /var/run/mysqld/mysqld.pid)
sudo systemctl start mysql
```

> [!warning] `--skip-grant-tables` désactive toute authentification
> Même avec `--skip-networking`, n'importe quel utilisateur local peut se connecter librement pendant cette procédure — à réserver aux cas de nécessité, et à refermer (redémarrage normal du service) le plus vite possible.

### Conflit avec MariaDB

```bash
dpkg -l | grep mariadb
sudo apt remove --purge mariadb-server mariadb-client mariadb-common
sudo rm -rf /var/lib/mysql/*
sudo apt update && sudo apt install -y mysql-server
```

> [!warning] Cette procédure détruit toutes les données MariaDB existantes
> Sauvegarder avec `mysqldump --all-databases` avant de vider `/var/lib/mysql/*` si des données doivent être conservées.

## Pour aller plus loin

L'architecture interne (instance, InnoDB, buffer pool) est couverte dans [[MySQL 01 — L'instance mysqld]] et suivantes. La configuration fine (dimensionnement du buffer pool, connexions, journalisation) et la sécurisation avancée (TLS, rôles, moindre privilège) restent des guides pratiques annoncés par la ressource source, non encore couverts dans ce vault.

Sources : [Installer MySQL sur Linux — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/installation/)
