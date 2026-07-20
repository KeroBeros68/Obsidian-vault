#linux #operations #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Toil (travail ingrat)** | Tâche opérationnelle manuelle, répétitive, automatisable, tactique, sans valeur durable et à croissance linéaire avec l'infrastructure — terminologie SRE (Google). |
| **SRE (Site Reliability Engineering)** | Discipline d'ingénierie appliquée à la fiabilité des systèmes, popularisée par Google, à l'origine du concept de toil et de la règle des 50%. |
| **Règle des 50% (Google)** | Recommandation SRE limitant le temps opérationnel d'une équipe à la moitié de son temps, le reste devant servir à l'ingénierie et à l'automatisation. |
| **Dette technique** | Ensemble des compromis techniques (délibérés ou involontaires) qui rendent un système plus difficile à maintenir, faire évoluer ou comprendre — métaphore financière de Ward Cunningham (1992). |
| **Quadrant de Fowler** | Classification de la dette technique selon deux axes (délibéré/inadvertant, prudent/imprudent), popularisée par Martin Fowler. |
| **TDR (Technical Debt Ratio)** | Indicateur = (coût de remédiation / coût de développement) × 100, utilisé pour quantifier la dette technique d'un projet. |
| **Code churn** | Part du code modifié peu de temps après son écriture ; un taux élevé signale du code écrit trop vite ou des exigences mal comprises au départ. |
| **Complexité cyclomatique** | Nombre de chemins d'exécution indépendants dans une fonction (≈ nombre de branches conditionnelles + 1), utilisé pour évaluer le besoin de refactoring. |
| **ADR (Architecture Decision Record)** | Document court consignant une décision architecturale, son contexte et ses conséquences, pour préserver la mémoire collective d'une équipe. |
| **Règle du scout** | Stratégie de remboursement de dette technique consistant à améliorer légèrement chaque fichier touché au passage, sans chantier dédié. |
| **Patch management** | Processus complet de gestion des correctifs : veille, inventaire, évaluation, test, déploiement, vérification et documentation — pas seulement l'application d'une mise à jour. |
| **CVSS (Common Vulnerability Scoring System)** | Score standardisé de sévérité d'une vulnérabilité, utilisé pour prioriser les correctifs de sécurité. |
| **MTTP (Mean Time To Patch)** | Délai moyen entre la publication d'une vulnérabilité et l'application du correctif correspondant. |
| **Déploiement canary** | Stratégie de déploiement appliquant un changement à un petit sous-ensemble de serveurs avant de l'étendre progressivement, pour limiter l'impact d'une régression. |
| **Rolling update** | Stratégie de déploiement appliquant un changement serveur par serveur, en le retirant puis le remettant dans le load balancer, pour un déploiement sans coupure. |
| **Blue-Green (déploiement)** | Stratégie de déploiement préparant un environnement complet (Green) avant de basculer le trafic depuis l'environnement courant (Blue), permettant un rollback quasi instantané. |
| **Baseline (état de référence)** | État validé et documenté d'un système — la définition de comment un serveur ou une ressource devrait être configuré. |
| **Dérive de configuration (configuration drift)** | Écart entre l'état réel d'un système et son état de référence, apparu au fil de modifications non tracées. |
| **Serveur snowflake** | Serveur ayant subi tant de modifications manuelles non documentées que plus personne ne sait comment le reproduire à l'identique. |
| **Fix Forward** | Stratégie de correction de dérive consistant à intégrer la modification constatée à l'état de référence plutôt qu'à l'annuler. |
| **Fix Backward** | Stratégie de correction de dérive consistant à rétablir l'état de référence antérieur (rollback), plutôt qu'à conserver la modification constatée. |
| **MTTD (Mean Time To Detect)** | Délai moyen entre l'apparition d'une dérive de configuration et sa détection. |
| **MTTR (Mean Time To Remediate)** | Délai moyen entre la détection d'une dérive et sa correction effective. |
| **IaC (Infrastructure as Code)** | Pratique consistant à décrire l'infrastructure sous forme de code versionné (Terraform, Ansible...), plutôt que par des actions manuelles — base de la prévention du toil et de la dérive de configuration. |
| **Capacity planning** | Mesurer la consommation actuelle des ressources (CPU, RAM, disque, réseau), observer la tendance et prévoir quand ajouter de la capacité, avant saturation. |
| **Runway** | Temps restant avant qu'une ressource soit saturée au rythme actuel de croissance : (capacité totale − utilisation actuelle) / taux de croissance. |
| **P95 (95ème percentile)** | Valeur en dessous de laquelle se situent 95% des mesures ; utilisée pour capturer les pics d'utilisation plutôt que la moyenne, qui les masque. |
| **OOM-Killer (Out-Of-Memory Killer)** | Mécanisme du noyau Linux qui tue des processus pour libérer de la mémoire quand la RAM (et le swap) sont épuisés. |
| **Load test / Stress test / Spike test** | Trois types de tests de charge : vérifier le comportement sous charge normale, trouver le point de rupture, tester la réaction à un pic soudain. |
