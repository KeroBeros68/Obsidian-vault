#bdd #mariadb #fondamentaux

## Un fork né d'une inquiétude sur la licence

MariaDB a été créé en 2009 par Michael « Monty » Widenius — créateur original de MySQL — et une partie de l'équipe MySQL historique, à la suite de l'annonce du rachat de Sun Microsystems (propriétaire de MySQL depuis 2008) par Oracle. Craignant qu'Oracle referme le développement de MySQL, Widenius a forké le code pour garantir qu'il reste disponible sous licence libre (GPL).

> [!info] Le nom vient d'une autre fille
> MySQL est nommé d'après *My*, la fille de Widenius. MariaDB est nommé d'après *Maria*, sa seconde fille.

La première version, MariaDB 5.1.38, est sortie le 29 octobre 2009 — avant même qu'Oracle ait officiellement finalisé le rachat de Sun. Le numéro de version n'était pas un hasard : il reproduisait exactement celui de MySQL 5.1.38, pour signaler une compatibilité totale au lancement.

## Deux organisations distinctes

Le développement est porté par deux entités : la **MariaDB Foundation** (à but non lucratif, garante du code ouvert et de la gouvernance communautaire) et **MariaDB Corporation** (entreprise commerciale, éditions Enterprise, support payant). MySQL, de son côté, appartient entièrement à Oracle depuis 2010.

## Compatibilité avec MySQL : forte mais décroissante

Historiquement, MariaDB visait une compatibilité binaire quasi totale avec MySQL : mêmes API clientes, mêmes connecteurs (PHP, Python, Java, ODBC...), fichiers de données compatibles. Cette compatibilité s'est réduite au fil des versions, à mesure que les deux projets ont divergé sur des fonctionnalités majeures.

| Aspect | MySQL | MariaDB |
|--------|-------|---------|
| Protocole client/connecteurs | Compatible (base commune) | Compatible (base commune) |
| Format JSON | Binaire compact (depuis 5.7) | `TEXT`/`BLOB` standard SQL |
| Réplication multi-source | Group Replication (consensus) | Galera Cluster (voir [[MariaDB 12 — Galera Cluster, réplication synchrone multi-maître]]) |
| GTID | Format `UUID:transaction_id` | Format `domaine-serveur-séquence`, **incompatible** avec celui de MySQL |
| Fonctionnalités temporelles | Absentes | Sequences, tables System-Versioned (voir [[MariaDB 06 — Sequences & tables System-Versioned]]) |
| Authentification par défaut | `caching_sha2_password` | `unix_socket` (local), `ed25519` (réseau) |
| Licence | GPL (Community), commerciale (Enterprise) | GPL uniquement |

> [!warning] La compatibilité n'est plus garantie au-delà de MySQL 5.6
> Un replica MySQL 5.6 ne peut pas répliquer depuis un source MariaDB 10.0+ : les formats GTID ont divergé. Dans l'autre sens (MariaDB en replica d'un MySQL ancien), une compatibilité partielle subsiste selon les versions, mais elle n'est plus l'axe de développement prioritaire des deux projets.

## Pourquoi choisir l'un ou l'autre aujourd'hui

MySQL reste la référence pour l'écosystème Oracle et les fonctionnalités les plus récentes du cloud managé (Group Replication, InnoDB Cluster). MariaDB se distingue par des fonctionnalités absentes de MySQL (Sequences, System Versioning, plus grande diversité de moteurs de stockage) et par Galera Cluster, une solution de haute disponibilité multi-maître mature et largement déployée en dehors de l'écosystème Oracle.

## Pour aller plus loin

Le cycle de versions et la politique de support à long terme de MariaDB sont détaillés dans [[MariaDB 07 — Versions & cycle de support (LTS)]].

Sources : [MariaDB — Wikipedia](https://en.wikipedia.org/wiki/MariaDB), [Incompatibilities and Feature Differences Between MariaDB and MySQL — MariaDB Documentation](https://mariadb.com/docs/release-notes/community-server/about/compatibility-and-differences/incompatibilities-and-feature-differences-between-mariadb-and-mysql-unmaint/incompatibilities-and-feature-differences-between-mariadb-10-4-and-mysql-8)
