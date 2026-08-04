#bdd #mariadb #réplication #avancé

## La solution HA phare de MariaDB, absente de MySQL

Galera Cluster est un plugin de réplication synchrone multi-maître, intégré à MariaDB depuis la version 10.1. Chaque nœud peut accepter des écritures, et un mécanisme de certification garantit que toutes les transactions sont appliquées dans le même ordre sur tous les nœuds. C'est l'équivalent fonctionnel le plus proche du Group Replication de MySQL (voir [[MySQL 30 — Semi-synchrone, Group Replication & InnoDB Cluster]]), mais conçu et développé indépendamment, avec une architecture différente.

> [!info] Réplication « virtuellement synchrone »
> Galera n'utilise pas un commit à deux phases classique (trop coûteux en performance). À la place, chaque transaction est diffusée sous forme de *write-set* à tous les nœuds au moment du `COMMIT`, puis **certifiée** sur chaque nœud avant d'être réellement appliquée — d'où le terme *virtuellement* synchrone plutôt que strictement synchrone.

## Réplication par certification : le mécanisme central

1. Une transaction s'exécute localement sur un nœud, sans verrou distribué.
2. Au `COMMIT`, le nœud rassemble les clés primaires des lignes modifiées dans un **write-set** et le diffuse à tous les nœuds du cluster via l'API **wsrep**.
3. Chaque nœud **certifie** le write-set : il vérifie qu'aucune transaction concurrente n'a modifié les mêmes lignes entre-temps.
4. Le write-set certifié est placé dans la file de réception du nœud, puis appliqué par un thread esclave Galera — de façon asynchrone par rapport à la certification elle-même.

```sql
SHOW STATUS LIKE 'wsrep_cluster_size';
SHOW STATUS LIKE 'wsrep_cluster_status';   -- Primary = quorum atteint
SHOW STATUS LIKE 'wsrep_local_state_comment';  -- Synced = à jour
```

> [!warning] Un conflit de certification échoue au COMMIT, pas avant
> Deux transactions concurrentes modifiant la même ligne sur deux nœuds différents ne sont détectées qu'au moment de la certification, après exécution complète de la transaction. La transaction perdante échoue avec une erreur de type deadlock (`ER_LOCK_DEADLOCK`) — l'application doit savoir réessayer une transaction échouée, comme pour tout deadlock InnoDB classique.

## Prérequis stricts

Galera impose des contraintes plus strictes que la réplication classique :

- **InnoDB uniquement** — les autres moteurs de stockage (MyISAM, Aria pour les tables utilisateur) ne répliquent pas correctement.
- **Clé primaire obligatoire** sur chaque table, pour permettre la certification.
- **3 nœuds minimum** (nombre impair recommandé) pour un quorum fiable en cas de partition réseau.
- Réseau à faible latence entre les nœuds — la certification distribuée est sensible aux temps de réponse.

## Quorum et split-brain

Si le cluster se scinde en deux groupes de nœuds suite à une panne réseau, seul le groupe disposant de la **majorité** des nœuds (le quorum) continue à accepter des écritures — le groupe minoritaire passe en `wsrep_cluster_status = Non-Primary` et refuse toute requête, empêchant un split-brain où deux sous-groupes accepteraient des écritures divergentes en parallèle.

```sql
SHOW STATUS LIKE 'wsrep_cluster_status';
-- Primary     → quorum atteint, écritures acceptées
-- Non-Primary → quorum perdu, lecture seule ou erreurs
```

> [!tip] Pourquoi un nombre impair de nœuds
> Avec 3 nœuds, une coupure réseau isole au maximum 1 nœud du groupe majoritaire de 2 — le quorum reste toujours déterminable. Avec un nombre pair (ex. 4), une coupure peut créer deux groupes de 2 nœuds strictement égaux, sans majorité claire, forçant l'arrêt des écritures des deux côtés.

## Streaming replication : les transactions volumineuses

Depuis Galera 4, la *streaming replication* fragmente une transaction très volumineuse (dépassant l'ancienne limite de 2 Go pour un write-set) en plusieurs fragments certifiés progressivement, plutôt qu'en un seul write-set monolithique au commit final.

## Pour aller plus loin

Une fois Galera en place, la sécurisation du cluster (comptes, TLS entre nœuds) suit les mêmes principes que pour une instance MariaDB classique — voir [[MariaDB 08 — Authentification (unix_socket, ed25519 & mysql_native_password)]] et [[MariaDB 13 — Sécurité & durcissement]].

Sources : [What is Galera Replication? — MariaDB Documentation](https://mariadb.com/docs/galera-cluster/readme/about-galera-replication), [Using MariaDB GTIDs with MariaDB Galera Cluster — MariaDB Documentation](https://mariadb.com/kb/en/using-mariadb-gtids-with-mariadb-galera-cluster/)
