#bdd #redis #installation #fondamentaux

## Installer Redis sur Debian/Ubuntu

```bash
sudo apt update
sudo apt install redis-server
sudo systemctl status redis-server
```

```bash
redis-cli PING
-- PONG
redis-cli INFO server | grep redis_version
```

> [!warning] Vérifier ce que le paquet installe réellement
> Depuis 2024, certaines distributions (Debian 13, Ubuntu 26.04 LTS, Fedora) ont remplacé `redis-server` par **Valkey** dans leurs dépôts par défaut, suite au changement de licence de Redis (voir [[Redis 01 — Licence, le fork Valkey]]). Vérifier avec `redis-cli INFO server` — une installation Valkey se présente généralement encore comme compatible protocole, mais le champ `redis_version`/`server` de la sortie `INFO` indique la provenance réelle.

## Installer depuis le dépôt officiel Redis

Pour une version plus récente que celle packagée par la distribution :

```bash
sudo apt install curl gpg lsb-release -y
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt update
sudo apt install redis
```

## Fichier de configuration

```bash
/etc/redis/redis.conf
```

```bash
sudo systemctl restart redis-server
```

> [!info] Structure du fichier, unités, CONFIG SET vs persistance
> La syntaxe complète de `redis.conf`, les unités de mémoire, l'`include`, et la différence entre `CONFIG SET` (à chaud) et `CONFIG REWRITE` (persistant) sont détaillées dans [[Redis 15 — Le fichier redis.conf & CONFIG REWRITE]].

> [!info] Un serveur en mémoire, pas de création de « base » au sens SQL
> Redis ne connaît pas de `CREATE DATABASE` : une instance expose par défaut 16 bases numérotées (`0` à `15`), sélectionnées avec `SELECT n` depuis le client — de simples espaces de clés isolés dans la même instance, sans schéma ni structure de table.

```bash
redis-cli
127.0.0.1:6379> SELECT 1
127.0.0.1:6379[1]> SET cle valeur
127.0.0.1:6379[1]> GET cle
```

## Pour aller plus loin

Le changement de licence de 2024 et la naissance du fork Valkey — un événement qui a redessiné tout l'écosystème — sont détaillés dans [[Redis 01 — Licence, le fork Valkey]].

Sources : [Redis quickstart — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-linux/)
