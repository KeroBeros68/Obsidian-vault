#bdd #redis #fondamentaux

## Redis n'est pas qu'un cache clé-valeur simple

Redis associe une clé (chaîne de caractères) à une valeur, mais cette valeur peut prendre plusieurs formes structurées — c'est ce qui distingue Redis d'un simple cache clé-valeur binaire comme Memcached.

## String : le type de base

```bash
SET session:42 "utilisateur_actif"
GET session:42

INCR compteur_vues        # Atomique, utile pour les compteurs
INCRBY stock 10
SETEX cache:page 60 "html_genere"   # Expire après 60 secondes
```

Une chaîne peut contenir du texte, du binaire brut, ou un objet sérialisé — jusqu'à 512 Mo par valeur.

## List : une séquence ordonnée

```bash
RPUSH file_attente "tache1" "tache2"   # Ajoute à droite
LPUSH file_attente "tache_prioritaire" # Ajoute à gauche
LRANGE file_attente 0 -1                # Liste tous les éléments
LPOP file_attente                       # Retire et retourne le premier élément
```

> [!tip] Une file d'attente FIFO/LIFO native
> `LPUSH` + `RPOP` (ou l'inverse) implémente directement une file FIFO. `BLPOP`/`BRPOP` ajoutent un blocage : le client attend qu'un élément soit disponible plutôt que de faire du polling actif — la base de nombreux systèmes de file de tâches légers.

## Hash : un objet à plusieurs champs

```bash
HSET utilisateur:42 nom "Dupont" email "dupont@example.com" age 34
HGET utilisateur:42 nom
HGETALL utilisateur:42
HINCRBY utilisateur:42 age 1
```

Un hash représente un enregistrement structuré (l'équivalent d'une ligne de table) sans avoir besoin de sérialiser/désérialiser un objet JSON complet à chaque lecture d'un seul champ.

## Set : une collection sans doublon ni ordre

```bash
SADD tags:article42 "redis" "bdd" "cache"
SISMEMBER tags:article42 "redis"    # 1 (existe)
SINTER tags:article42 tags:article7  # Intersection entre deux sets
SCARD tags:article42                 # Nombre d'éléments
```

Les opérations d'ensemble (`SINTER`, `SUNION`, `SDIFF`) s'exécutent en temps quasi constant côté Redis, ce qui les rend idéales pour des calculs qui seraient coûteux à répéter côté application (tags communs, recommandations, permissions).

## Sorted Set : un ensemble trié par score

```bash
ZADD classement 1500 "joueur_a" 2300 "joueur_b" 1800 "joueur_c"
ZRANGE classement 0 -1 WITHSCORES     # Du plus petit au plus grand score
ZREVRANGE classement 0 2               # Top 3
ZINCRBY classement 100 "joueur_a"      # Incrémente le score
ZRANK classement "joueur_a"            # Position dans le classement
```

> [!info] Le cas d'usage emblématique : le classement (leaderboard)
> Un Sorted Set maintient automatiquement l'ordre par score à chaque insertion, avec des opérations de lecture par rang ou par score en O(log N) — exactement la structure dont a besoin un classement de jeu vidéo, un système de priorité, ou un plafond de limitation de débit (*rate limiting*) basé sur une fenêtre temporelle.

## Choisir le bon type

| Besoin | Type Redis |
|--------|-----------|
| Compteur, cache d'une valeur simple, verrou | String |
| File d'attente, historique récent (N derniers éléments) | List |
| Objet structuré (profil, configuration) | Hash |
| Appartenance, tags, permissions | Set |
| Classement, priorité, fenêtre glissante | Sorted Set |

## Pour aller plus loin

Au-delà de ces cinq types fondamentaux, Redis propose des structures spécialisées — Streams, Bitmaps, HyperLogLog, géospatial — couvertes dans [[Redis 04 — Types de données avancés]].

Sources : [Redis data types — Redis Documentation](https://redis.io/docs/latest/develop/data-types/), [Redis sorted sets — Redis Documentation](https://redis.io/docs/latest/develop/data-types/sorted-sets/)
