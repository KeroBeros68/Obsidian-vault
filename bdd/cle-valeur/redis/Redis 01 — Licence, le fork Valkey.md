#bdd #redis #fondamentaux

## Un changement de licence qui a redessiné l'écosystème

Jusqu'à la version 7.2, Redis était distribué sous licence **BSD 3-Clause**, une licence open source permissive sans restriction d'usage commercial. En mars 2024, Redis Inc. a changé la licence des futures versions vers un choix entre **RSALv2** (*Redis Source Available License*) et **SSPLv1** (*Server Side Public License*) — deux licences *source-available*, non reconnues comme open source par l'OSI (*Open Source Initiative*).

| Version | Licence | Statut OSI |
|---------|---------|------------|
| ≤ 7.2 | BSD 3-Clause | Open source |
| 7.4.x - 7.8.x | RSALv2 **ou** SSPLv1 (au choix) | Source-available, non-OSI |
| ≥ 8.0.0 | RSALv2, SSPLv1, **ou AGPLv3** (au choix) | AGPLv3 est OSI-approuvé |

> [!warning] Impact concret du changement
> RSALv2 et SSPLv1 interdisent de commercialiser Redis en tant que service managé proposé à des tiers, sans en reverser le code (SSPLv1 exige même la publication du code de la couche de gestion). Pour l'immense majorité des utilisateurs — une application qui utilise Redis en interne, sans le revendre comme service — l'impact réel est nul : le code s'utilise, se modifie et se déploie sans restriction.

## La réponse de la communauté : Valkey

En réaction, un groupe d'ingénieurs de plusieurs grandes entreprises (AWS, Google Cloud, Oracle, Ericsson...) a créé **Valkey**, un fork démarré à partir de Redis 7.2.4 — la dernière version sous licence BSD — placé sous la gouvernance de la **Linux Foundation** (annonce du 28 mars 2024) et maintenu sous licence BSD 3-Clause.

> [!info] Une gouvernance ouverte, pas un simple fork de circonstance
> Valkey n'est pas un fork isolé : il rassemble une bonne partie des anciens mainteneurs de Redis et bénéficie du soutien de fournisseurs cloud majeurs. Amazon ElastiCache utilise Valkey par défaut, et plusieurs distributions Linux (Debian 13, Ubuntu 26.04 LTS, Fedora 42, Arch) ont basculé leur paquet par défaut de `redis` vers `valkey`.

## Redis 8 : un retour partiel vers l'open source

Face aux critiques, Redis Inc. a ajouté l'**AGPLv3** (licence open source approuvée par l'OSI) comme troisième choix possible à partir de la version 8.0 (mai 2025), rebaptisant l'édition communautaire **Redis Open Source**. Un utilisateur peut désormais choisir de placer son usage sous AGPLv3 plutôt que sous les licences source-available.

> [!warning] AGPLv3 a sa propre clause réseau
> AGPLv3 impose que si le logiciel modifié tourne en tant que service accessible par réseau, les utilisateurs de ce service doivent pouvoir accéder au code source des modifications. C'est une contrainte différente de RSALv2/SSPLv1, mais qui reste plus stricte qu'une licence permissive comme BSD — à évaluer selon le contexte d'usage (interne vs service exposé).

## Redis vs Valkey aujourd'hui : compatibilité quasi totale

Valkey ayant démarré comme copie exacte de Redis 7.2.4, la compatibilité de protocole et de commandes reste très large entre les deux projets — la plupart du contenu de ce module (types de données, persistance, réplication, Sentinel, Cluster) s'applique de façon quasi identique aux deux. Les deux projets ont depuis évolué indépendamment, avec des fonctionnalités propres à chacun apparues après le fork (mars 2024).

> [!tip] Quel choix pour un nouveau projet
> Pour un usage interne standard (cache, sessions, files de messages), la question de licence importe peu : Redis Open Source (choix AGPLv3) et Valkey (BSD) conviennent tous les deux. Pour un service managé proposé à des tiers, ou par principe pour une licence permissive sans ambiguïté, Valkey évite toute question juridique.

## Pour aller plus loin

L'architecture interne — un seul thread pour l'exécution des commandes, mais pas pour tout le reste — est détaillée dans [[Redis 02 — Architecture, event loop & threads réseau]].

Sources : [The Redis License Timeline: BSD to SSPL to AGPL — redisvsmemcached.com](https://redisvsmemcached.com/redis-license-timeline/), [Redis licenses — Redis Documentation](https://redis.io/legal/licenses/), [Linux Foundation Launches Open Source Valkey Community](https://www.linuxfoundation.org/press/linux-foundation-launches-open-source-valkey-community), [Valkey · An Investment in Open Source](https://valkey.io/blog/valkey-investment-in-open-source/)
