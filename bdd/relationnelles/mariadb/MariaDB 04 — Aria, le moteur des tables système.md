#bdd #mariadb #avancé

## Le successeur crash-safe de MyISAM

Aria est un moteur de stockage développé par l'équipe MariaDB pour remplacer MyISAM : même simplicité et mêmes performances en lecture, mais avec une protection contre les crashes que MyISAM n'a jamais offerte. Depuis MariaDB 10.4, **toutes les tables système** (`mysql.global_priv`, `mysql.db`, etc. — voir [[MariaDB 02 — Bases de données & bases système]]) ainsi que les tables temporaires internes utilisent Aria plutôt que MyISAM.

```sql
CREATE TABLE logs_lecture (
  id INT PRIMARY KEY,
  message TEXT
) ENGINE=Aria;
```

## TRANSACTIONAL : la sécurité en cas de crash

Par défaut, une table Aria n'est pas transactionnelle au sens InnoDB. L'option `TRANSACTIONAL=1` active un journal de transactions propre à Aria, qui synchronise les écritures à la fin de chaque instruction :

```sql
CREATE TABLE logs_critiques (
  id INT PRIMARY KEY,
  message TEXT
) ENGINE=Aria TRANSACTIONAL=1;
```

> [!info] Un coût mesuré
> Activer `TRANSACTIONAL=1` protège contre la perte de données en cas d'arrêt brutal du serveur, au prix d'un léger surcoût en écriture et jusqu'à 6 octets supplémentaires par ligne/clé. C'est un compromis intermédiaire entre MyISAM (aucune protection) et InnoDB (transactions ACID complètes).

## Format de page et cache

Aria utilise par défaut le format `PAGE`, plus proche d'InnoDB que du format historique de MyISAM, ce qui améliore la mise en cache — en particulier pour les requêtes `GROUP BY` et `DISTINCT`.

| Paramètre | Rôle |
|-----------|------|
| `aria_pagecache_buffer_size` | Taille du cache de pages (index + données) — l'équivalent du buffer pool InnoDB pour Aria |
| `aria_block_size` | Taille de bloc (8192 octets par défaut), affecte l'efficacité des lookups |

```sql
SHOW VARIABLES LIKE 'aria_pagecache_buffer_size';
SET GLOBAL aria_pagecache_buffer_size = 256*1024*1024;  -- 256 Mo
```

> [!tip] Pour émuler l'ancien comportement MyISAM
> Spécifier `ROW_FORMAT=FIXED` ou `ROW_FORMAT=DYNAMIC` à la création d'une table Aria reproduit le comportement de stockage de MyISAM, plutôt que le format `PAGE` par défaut.

## Quand choisir Aria pour ses propres tables

Aria convient à des données en lecture intensive qui n'ont pas besoin des garanties transactionnelles complètes d'InnoDB (verrous ligne, MVCC) : tables de logs applicatifs, tables de référence peu modifiées, tables temporaires volumineuses créées manuellement. Pour toute donnée transactionnelle (comptes, commandes, stocks), InnoDB reste le choix par défaut — voir [[MariaDB 05 — InnoDB dans MariaDB]].

## Pour aller plus loin

InnoDB, partagé avec MySQL mais avec quelques divergences de réglage propres à MariaDB, est détaillé dans [[MariaDB 05 — InnoDB dans MariaDB]].

Sources : [Aria Storage Engine — MariaDB Documentation](https://mariadb.com/docs/server/server-usage/storage-engines/aria/aria-storage-engine), [MDEV-20002 — Create system tables as aria instead of myisam](https://jira.mariadb.org/browse/MDEV-20002)
