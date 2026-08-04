#bdd #mariadb #fondamentaux

## Une philosophie de release opposée au modèle « evergreen » de MySQL

MariaDB choisit délibérément la stabilité plutôt que le rythme de nouveautés : une fois une série publiée comme stable, ses versions mineures suivantes ne doivent pas introduire de nouvelles fonctionnalités ni de régressions de compatibilité — uniquement des correctifs de bugs et de sécurité.

> [!info] L'opposé du modèle MySQL 8.0
> Le projet MariaDB décrit lui-même cette approche comme « l'opposé du modèle evergreen de MySQL 8.0 », où de nouvelles fonctionnalités majeures et des incompatibilités peuvent apparaître dans une branche qualifiée de stable. MariaDB, à l'inverse, garde des releases LTS figées fonctionnellement pendant toute leur durée de support.

## Cycle de release et séries LTS

Une série LTS (*Long Term Support*) est publiée périodiquement, en coordination avec les cycles des grandes distributions Linux — MariaDB 10.11 a par exemple été calée pour être la version incluse dans Debian 12.

| Série | Statut (2026-07) | Support jusqu'à |
|-------|-------------------|-------------------|
| 10.6 LTS | Support communautaire terminé (2026-07-06) | — |
| 10.11 LTS | Supportée | Selon calendrier officiel |
| 11.8 LTS | Supportée | ~2028 |
| 12.3 LTS | Dernière LTS publiée | ~2029 |
| 13.x | Développement / preview | — |

> [!warning] Vérifier la date exacte avant toute décision de production
> Ces dates évoluent : consulter la page officielle [MariaDB Server — All releases](https://mariadb.org/mariadb/all-releases/) avant de figer un choix de version, plutôt que de se fier à un tableau figé dans une fiche.

Depuis la série 12.x, MariaDB adopte une convention fixe : **la version `.3` de chaque série majeure devient la release LTS**, simplifiant le repérage de la version à privilégier en production.

## Support communautaire vs Enterprise

Une série LTS communautaire reçoit correctifs de bugs et de sécurité pendant environ 3 ans (5 ans pour certaines séries plus anciennes comme 11.4). Un abonnement MariaDB Enterprise prolonge ce support de 2 ans supplémentaires, avec des correctifs de sécurité critiques distribués en code source pendant 2 années de plus au-delà, en best-effort.

## Choisir une série pour un nouveau projet

- **Débuter un projet aujourd'hui** : privilégier la dernière série LTS publiée (voir le tableau ci-dessus, à vérifier).
- **Suivre la version packagée par sa distribution Linux** : accepter une version plus ancienne mais sans gérer de dépôt tiers (voir [[MariaDB 00 — Installation]]).
- **Éviter les séries de développement/preview** en production : elles peuvent encore introduire des changements de comportement avant la stabilisation.

## Pour aller plus loin

Le modèle d'authentification par défaut, distinct de celui de MySQL, est détaillé dans [[MariaDB 08 — Authentification (unix_socket, ed25519 & mysql_native_password)]].

Sources : [MariaDB 10.11 is LTS — MariaDB.org](https://mariadb.org/mariadb-10-11-is-lts/), [MariaDB Server — All releases — MariaDB.org](https://mariadb.org/mariadb/all-releases/), [MariaDB Community Server 11.8 LTS is Now Available — MariaDB](https://mariadb.com/resources/blog/latest-lts-version-of-mariadb-community-server-11-8-is-now-available/)
