#bdd #redis #avancé

## RDB : un instantané compact à intervalles réguliers

RDB (*Redis Database*) sauvegarde périodiquement l'intégralité du jeu de données dans un fichier binaire unique (`dump.rdb`), à des points de sauvegarde configurables.

```bash
# redis.conf : sauvegarde si au moins 1000 clés ont changé en 60 secondes
save 60 1000
save 300 10
save 3600 1
```

```bash
SAVE       # Bloquant, dans le thread principal — à éviter en production
BGSAVE     # Non bloquant, via un processus enfant
```

## Comment fonctionne BGSAVE : fork et copy-on-write

1. Redis appelle `fork()` : un processus enfant est créé, copie exacte de l'espace mémoire du parent.
2. Le processus enfant écrit tout le jeu de données dans un fichier RDB temporaire.
3. Une fois l'écriture terminée, le fichier temporaire remplace l'ancien `dump.rdb`.

> [!info] Pourquoi le fork ne duplique pas réellement toute la RAM
> Grâce au *copy-on-write* du noyau Linux, les pages mémoire ne sont dupliquées que si le processus parent les modifie pendant que l'enfant les lit encore — sur un serveur peu modifié pendant le snapshot, le coût mémoire réel du fork reste faible. Sur un serveur à très fort taux d'écriture et gros volume de données, le fork peut néanmoins bloquer brièvement le thread principal (de quelques millisecondes à une seconde sur un jeu de données très volumineux).

## Avantages et limites de RDB

| Avantages | Limites |
|-----------|---------|
| Fichier compact, idéal pour l'archivage et le transfert (S3, sauvegarde distante) | Perte de données possible entre deux points de sauvegarde (jusqu'à plusieurs minutes) |
| Redémarrage rapide même sur un gros volume de données | `fork()` peut être coûteux en CPU/mémoire sur un jeu de données volumineux à fort débit d'écriture |
| Aucune E/S disque dans le thread principal (déléguée au processus enfant) | Pas de garantie de durabilité stricte en cas de crash entre deux snapshots |
| Permet une resynchronisation partielle après un redémarrage de replica | — |

> [!warning] RDB seul ne convient pas si la tolérance à la perte de données est faible
> Entre deux points de sauvegarde, un crash serveur perd toutes les écritures intervenues depuis le dernier snapshot. Pour une durabilité plus fine, combiner RDB avec AOF (voir [[Redis 08 — Persistance AOF (append-only file)]]) ou activer AOF seul avec une politique de `fsync` fréquente.

## Sauvegarder un fichier RDB en toute sécurité

```bash
# Copier le fichier pendant que le serveur tourne : sans danger
cp /var/lib/redis/dump.rdb /backup/dump-$(date +%F).rdb
```

Le fichier RDB n'est jamais modifié une fois produit : il est renommé atomiquement à la fin de son écriture, ce qui rend sa copie sans risque à tout moment, y compris pendant que le serveur continue de fonctionner.

## Pour aller plus loin

Pour une durabilité plus stricte (perte de données limitée à moins d'une seconde), le mécanisme AOF est détaillé dans [[Redis 08 — Persistance AOF (append-only file)]].

Sources : [Redis persistence — Redis Documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)
