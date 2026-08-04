#bdd #redis #fondamentaux

## Où vit le fichier, et pourquoi il est indispensable

Redis démarre sans problème avec sa configuration par défaut intégrée, mais ce mode ne convient qu'au test — toute instance de production doit démarrer avec un fichier explicite :

```bash
redis-server /etc/redis/redis.conf
```

Sur Debian/Ubuntu (voir [[Redis 00 — Installation]]), ce fichier se trouve dans `/etc/redis/redis.conf` et le service systemd le référence déjà par défaut.

> [!info] Redis 8 : deux fichiers plutôt qu'un
> Depuis Redis 8, la distribution fournit `redis.conf` (le seul cœur serveur) et `redis-full.conf` (serveur + tous les composants intégrés — recherche, séries temporelles, structures probabilistes, JSON). `redis-full.conf` commence par une directive `include redis.conf` puis ajoute quatre `loadmodule`. Utiliser `redis-full.conf` pour bénéficier de l'ensemble des types avancés couverts en [[Redis 04 — Types de données avancés]].

## Syntaxe du fichier

Chaque ligne est une directive au format `mot-clé argument1 argument2 ...` :

```conf
# Ceci est un commentaire
port 6379
bind 127.0.0.1 10.0.1.5
requirepass "mot de passe avec espaces"   # guillemets nécessaires si espaces
maxmemory 2gb
```

> [!info] Unités de mémoire acceptées
> `maxmemory` et directives similaires acceptent `1k`/`1kb`, `1m`/`1mb`, `1g`/`1gb` (insensible à la casse) — `1gb` et `1GB` sont strictement équivalents.

## dir : le répertoire de travail, source classique de confusion

```conf
dir /var/lib/redis
```

`dir` fixe l'endroit où Redis écrit ses fichiers `dump.rdb` et son répertoire AOF (voir [[Redis 07 — Persistance RDB (snapshots)]] et [[Redis 08 — Persistance AOF (append-only file)]]) — **relatif** à ce répertoire, pas au répertoire depuis lequel `redis-server` a été lancé.

> [!warning] Un `dir` mal positionné écrit les fichiers ailleurs que prévu
> Si `dir` n'est pas explicitement absolu, Redis résout les chemins relatifs (`dbfilename`, `appenddirname`) par rapport au répertoire courant du processus au démarrage, qui n'est pas nécessairement celui attendu (dépend de la façon dont le service est lancé). Toujours fixer `dir` à un chemin absolu en production.

## include : fragmenter la configuration

```conf
include /etc/redis/base-commune.conf
include /etc/redis/conf.d/*.conf
```

Permet de partager un socle commun entre plusieurs instances tout en personnalisant certains réglages par serveur. Les chemins avec jokers sont inclus par ordre alphabétique ; un motif qui ne correspond à aucun fichier est simplement ignoré, sans erreur.

> [!warning] L'ordre des include détermine qui gagne
> Redis retient toujours la **dernière** ligne traitée pour une directive donnée. Placer les `include` en tête de fichier si on veut que les réglages qui suivent dans le fichier principal aient le dernier mot ; les placer en fin de fichier si on veut au contraire que les fichiers inclus puissent surcharger le reste.

## Passer des paramètres en ligne de commande

```bash
redis-server --port 6380 --replicaof 127.0.0.1 6379
```

Même format que dans le fichier, avec un préfixe `--` — pratique pour des tests ponctuels, à éviter comme méthode de configuration permanente en production.

## CONFIG GET / CONFIG SET : lire et modifier à chaud

```bash
CONFIG GET maxmemory*        # Motif glob : maxmemory, maxmemory-policy, maxmemory-samples...
CONFIG SET maxmemory 500mb
```

La plupart des directives sont modifiables à chaud sans redémarrage — mais pas toutes : certaines (le port d'écoute principal, par exemple) exigent un redémarrage complet du serveur.

> [!warning] CONFIG SET ne touche jamais le fichier redis.conf
> Contrairement à `SET PERSIST` en MySQL (qui écrit automatiquement dans `mysqld-auto.cnf`, voir [[MySQL 11 — Configuration (InnoDB, mémoire, connexions & logging)]]), un `CONFIG SET` Redis ne modifie que l'état en mémoire du serveur. Un redémarrage sans action supplémentaire fait **perdre** ce changement et revenir à ce que dit `redis.conf`.

## CONFIG REWRITE : rendre un changement permanent

```bash
CONFIG SET maxmemory 500mb
CONFIG REWRITE
```

`CONFIG REWRITE` relit le fichier de configuration d'origine et le met à jour pour refléter l'état courant du serveur, de façon délibérément conservatrice :

- Une directive déjà présente dans le fichier est réécrite à la même ligne.
- Une directive absente du fichier mais laissée à sa valeur par défaut n'est **pas** ajoutée.
- Une directive absente du fichier et modifiée à une valeur non-défaut est ajoutée en fin de fichier.
- Les commentaires et la structure générale du fichier sont préservés autant que possible.

> [!warning] CONFIG REWRITE échoue sans fichier de configuration au démarrage
> Un serveur lancé sans argument de fichier (`redis-server` seul, ou uniquement avec des `--options` en ligne de commande) n'a rien à réécrire — `CONFIG REWRITE` renvoie une erreur. Toujours démarrer avec un fichier `redis.conf` explicite en production, ne serait-ce que pour que `CONFIG REWRITE` fonctionne.

## Exemple minimal commenté

```conf
port 6379
bind 127.0.0.1 10.0.1.5
protected-mode yes
requirepass "MotDePasseFort2026!"

dir /var/lib/redis
dbfilename dump.rdb
appendonly yes
appendfsync everysec

maxmemory 2gb
maxmemory-policy allkeys-lru

logfile /var/log/redis/redis.log
```

## Pour aller plus loin

Cette fiche complète l'installation (voir [[Redis 00 — Installation]]) et sert de référence transversale à tout le reste du module — les fiches persistance, réplication, Sentinel et sécurité renvoient chacune à des directives spécifiques de ce même fichier.

Sources : [Redis configuration — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/config/), [CONFIG REWRITE — Redis Documentation](https://redis.io/docs/latest/commands/config-rewrite/)
