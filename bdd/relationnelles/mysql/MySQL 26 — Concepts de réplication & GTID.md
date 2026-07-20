#bdd #mysql #réplication #avancé

## Le problème : un serveur unique est un SPOF

Un MySQL seul est un point de défaillance unique (SPOF) : panne disque, crash système, maintenance planifiée — l'application est indisponible tant que le serveur n'est pas relancé. La **réplication** résout ce problème en copiant en continu les modifications vers un ou plusieurs **replicas**, qui peuvent prendre le relais en cas de panne, absorber les lectures, ou servir de base pour les sauvegardes.

> [!warning] La réplication n'est pas une sauvegarde
> La réplication propage tout du source vers le replica : les `INSERT` mais aussi les `DELETE`, les `DROP TABLE` et les erreurs humaines. Un `DROP DATABASE` sur le source est immédiatement répliqué. Pour se protéger des erreurs humaines et de la corruption, des sauvegardes avec archivage binlog et PITR restent indispensables — voir [[MySQL 12 — Sauvegarde et restauration (mysqldump, MySQL Shell, XtraBackup, PITR)]].

> [!info] Version utilisée
> Ce guide utilise MySQL 8.4 LTS. Les termes *source*/*replica* remplacent officiellement *master*/*slave* ; les *tagged GTIDs* sont introduits ; le clone plugin simplifie l'initialisation des replicas.

## Les trois modèles de réplication MySQL

**Réplication classique (source → replica, asynchrone)** — le modèle le plus courant. Le source écrit les modifications dans le binary log (voir [[MySQL 07 — Binary log — réplication & PITR]]). Le replica lit ce binary log, le copie dans son relay log local, puis l'applique. Le source n'attend pas la confirmation du replica avant de valider un `COMMIT` : simple et performant, mais quelques transactions peuvent être perdues si le source crashe avant que le replica n'ait reçu les derniers binlogs.

**Semi-synchrone (plugin `rpl_semi_sync`)** — le source attend la confirmation du replica avant de retourner le `COMMIT` au client. Garantit qu'au moins un replica a reçu les données (RPO quasi nul), au prix d'une latence réseau supplémentaire par `COMMIT`.

**Group Replication (consensus Paxos, multi-primaire)** — plusieurs nœuds forment un groupe qui maintient la cohérence par consensus distribué. Chaque nœud peut accepter des écritures (*multi-primary*) ou un seul (*single-primary*). Détection de pannes et failover automatiques.

| Critère | Asynchrone | Semi-synchrone | Group Replication |
|---------|-----------|----------------|---------------------|
| Complexité | Simple | Modérée | Élevée |
| RPO | Quelques transactions | Quasi nul | Nul (consensus) |
| RTO | Manuel (failover) | Manuel | Automatique |
| Latence COMMIT | Aucun impact | +1 aller-retour réseau | +consensus |
| Nombre de nœuds | 2+ | 2+ | 3+ (impair recommandé) |
| Cas d'usage | Lecture distribuée, backup | Données critiques, finance | Haute disponibilité automatisée |

> [!tip] Règle pratique
> Commencer par la réplication GTID asynchrone. Passer en semi-synchrone pour les données critiques, et en Group Replication quand le failover automatique est indispensable — voir [[MySQL 30 — Semi-synchrone, Group Replication & InnoDB Cluster]].

## GTID : la brique essentielle

Un **GTID** (*Global Transaction Identifier*) identifie de manière unique chaque transaction validée sur un serveur MySQL. Format : `source_uuid:transaction_id`, par exemple `3E11FA47-71CA-11E1-9E33-C80AA9429562:23`.

- `source_uuid` : identifiant unique du serveur qui a créé la transaction (variable `server_uuid`)
- `transaction_id` : numéro séquentiel incrémenté à chaque `COMMIT`

L'ensemble des GTID exécutés sur un serveur s'appelle le **GTID set** (visible dans `gtid_executed`). Lors de la réplication, le replica sait exactement quelles transactions il a déjà appliquées — il ne reçoit jamais un doublon.

> [!info] Tagged GTIDs en MySQL 8.4
> MySQL 8.4 introduit les *tagged GTIDs* au format `UUID:TAG:NUMBER`. Les tags permettent de catégoriser les transactions (opérations système vs applicatives) et simplifient le filtrage dans les scénarios multi-sources. Fonctionnalité optionnelle, ne modifie pas le comportement par défaut.

| Critère | Position binlog (ancien) | GTID (recommandé) |
|---------|---------------------------|----------------------|
| Failover | Recalculer la position sur le nouveau source, complexe et fragile | Le replica retrouve automatiquement sa position par le GTID set |
| Point de reprise | Dépend du fichier + offset dans le binlog | Identifiant global, indépendant du fichier |
| Multi-source | Très complexe | Géré nativement (chaque source a son UUID) |
| Détection de doublons | Impossible | Le replica refuse les GTID déjà appliqués |
| Recommandation | Legacy, pour compatibilité | Standard moderne, recommandé |

## Pour aller plus loin

La mise en place concrète d'un couple source-replica en GTID est détaillée dans [[MySQL 27 — Mettre en place une réplication source-replica (GTID)]].

Sources : [Réplication MySQL : source-replica, GTID et haute disponibilité — Stéphane Robert](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/mysql/replication/)
