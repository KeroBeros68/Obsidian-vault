#bdd #mysql #innodb #performance #avancé

## Le cache de pages d'InnoDB

Le **buffer pool** est la zone de mémoire principale d'InnoDB — un cache de pages (16 Ko chacune) qui stocke à la fois les **données** et les **index** des tables InnoDB. Quand MySQL lit une ligne, il charge la page qui la contient dans le buffer pool ; les lectures suivantes de cette même page sont servies depuis la mémoire, sans accès disque.

> [!tip] Analogie : le plan de travail du cuisinier
> Plus le plan de travail est grand, plus on garde d'ingrédients (pages de données) à portée de main sans retourner au frigo (le disque). Un buffer pool trop petit force MySQL à aller constamment sur disque, ce qui ralentit toutes les opérations.

```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
-- 134217728 octets = 128 Mo (valeur par défaut)
```

> [!warning] 128 Mo est adapté à un lab, pas à la production
> Sur un serveur dédié à MySQL, la documentation Oracle indique qu'on peut aller jusqu'à 80% de la RAM physique pour le buffer pool. En pratique, **50 à 80%** de la mémoire disponible est un bon point de départ selon les autres services présents sur la machine — sur un serveur de 4 Go dédié à MySQL, `innodb_buffer_pool_size = 2G` à `3G` est raisonnable.

## Inspecter l'utilisation du buffer pool

```sql
SHOW ENGINE INNODB STATUS \G
```

Section `BUFFER POOL AND MEMORY` :

| Métrique | Signification |
|----------|-------------------|
| Buffer pool size | Nombre total de pages (ex. 8192 × 16 Ko = 128 Mo) |
| Free buffers | Pages libres, inutilisées |
| Database pages | Pages contenant des données en cache |
| Modified db pages | Pages modifiées en mémoire mais pas encore écrites sur disque (*dirty pages*) |

> [!warning] `Free buffers` à 0 signale un buffer pool sous pression
> Quand les pages libres atteignent zéro, InnoDB doit **évincer** des pages anciennes pour en charger de nouvelles — c'est exactement le moment où les performances se dégradent, puisque des pages utiles sont expulsées du cache pour faire de la place.

## Cas particuliers

> [!info] Le dimensionnement précis est affaire de configuration, pas d'architecture
> Cette fiche explique ce qu'est le buffer pool et comment lire son état ; le réglage fin (`innodb_buffer_pool_instances`, ratio exact selon la charge) relève d'un guide de configuration dédié, non couvert dans ce vault pour l'instant.

## Pour aller plus loin

Les autres structures internes d'InnoDB (journalisation, tablespaces) sont dans [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]].

Sources : [Découvrir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/decouvrir-mysql/)
