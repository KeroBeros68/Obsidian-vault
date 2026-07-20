#bdd #mysql #fondamentaux #versions

## Deux tracks parallèles depuis 2023

Oracle publie MySQL selon deux tracks distincts :

| | LTS (Long Term Support) | Innovation |
|---|----------------------------|----------------|
| Version actuelle | 8.4 (avril 2024) | 9.0, 9.1, 9.2... |
| Positionnement | Stabilité fonctionnelle | Accès rapide aux nouveautés |
| Support | Long (5 ans Premier + 3 ans Extended) | Cycle court, jusqu'à la version suivante |
| Stabilité API | Pas de suppression de fonctionnalités | Fonctions retirées ou renommées entre versions |
| Cas d'usage | Production stable, standardisation | Équipes prêtes à suivre un rythme rapide |
| Fréquence | Patches trimestriels (8.4.x) | Nouvelle version tous les ~3 mois |

> [!tip] Recommandation pour un nouveau déploiement
> **MySQL 8.4 LTS** pour tout nouveau déploiement en production. Les deux tracks sont production-grade, mais LTS offre une stabilité fonctionnelle et un support long mieux adaptés à la plupart des organisations. Innovation convient aux équipes prêtes à suivre un rythme de mise à jour trimestriel pour accéder aux dernières fonctionnalités.

> [!warning] MySQL 8.0 est en fin de vie
> MySQL 8.0 a atteint sa fin de vie (EoL) en avril 2026 — Oracle ne publie plus de correctifs de sécurité pour l'édition communautaire au-delà de cette date (le numéro de version exact du dernier correctif varie selon les sources, 8.0.42 ou 8.0.46 selon le relevé ; le fait à retenir est la date d'EOL elle-même, confirmée). Pour tout nouveau déploiement, MySQL 8.4 LTS s'impose.

## Changements notables de MySQL 8.4 LTS par rapport à 8.0

- Le binary log est activé par défaut (plus besoin de `--log-bin` explicite) — voir [[MySQL 07 — Binary log — réplication & PITR]].
- Le plugin `mysql_native_password` n'est plus activé par défaut ; `caching_sha2_password` devient la référence (l'ancien plugin reste disponible mais déprécié, supprimé en MySQL 9.0).
- `innodb_log_file_size`/`innodb_log_files_in_group` sont remplacées par `innodb_redo_log_capacity` — voir [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]].
- `CHANGE MASTER TO` devient `CHANGE REPLICATION SOURCE TO` ; `SHOW SLAVE STATUS` devient `SHOW REPLICA STATUS` — le terme *slave* est remplacé par *replica* dans toute la documentation et les commandes.

## Arborescence d'une installation MySQL 8.4 (Debian 12)

```
/etc/mysql/
├── my.cnf                          → fichier principal
├── conf.d/mysql.cnf                 → options du client mysql
└── mysql.conf.d/mysqld.cnf          → configuration du serveur mysqld

/var/lib/mysql/                      → datadir (toutes les données)
├── #innodb_redo/                    → redo log
├── ibdata1                          → tablespace système
├── mysql.ibd                        → data dictionary
├── binlog.000001                    → binary log
├── undo_001, undo_002                → undo tablespaces (MVCC, rollback)
└── labdb/                           → un répertoire par base créée

/var/log/mysql/
├── error.log                        → erreurs et démarrage
└── mysql-slow.log                   → requêtes lentes (si activé)

/var/run/mysqld/mysqld.sock          → socket Unix
```

| Fichier/Répertoire | Criticité |
|------------------------|---------------|
| `/var/lib/mysql/` | Critique — toute l'intégrité de la base en dépend, à sauvegarder |
| `ibdata1` | Critique — ne jamais supprimer |
| `mysql.ibd` | Critique — indispensable au démarrage (data dictionary) |
| `binlog.*` | Élevée — nécessaire pour la restauration et la réplication |
| `auto.cnf` | Moyenne — ne jamais copier tel quel sur un réplica (voir [[MySQL 06 — InnoDB — redo log, doublewrite buffer & tablespaces]]) |

## Vocabulaire essentiel

Le vocabulaire complet du module (instance, InnoDB, buffer pool, GTID...) est rassemblé dans [[MySQL — Glossaire]].

## Cas particuliers

> [!info] Rester informé au-delà de ce module
> Les tracks de version évoluent (nouvelles versions Innovation, prochaine génération LTS) — vérifier `SELECT VERSION();` sur une instance existante et consulter les notes de version officielles avant toute décision de mise à niveau.

Sources : [Découvrir MySQL — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/decouvrir-mysql/), vérification complémentaire sur la date de fin de vie de MySQL 8.0.
