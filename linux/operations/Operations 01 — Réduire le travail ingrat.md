#linux #operations #sre #toil

## Qu'est-ce que le travail ingrat (toil)

Le **travail ingrat** (*toil* en littérature SRE — Site Reliability Engineering, les équipes Google chargées de la fiabilité des systèmes) désigne les tâches opérationnelles manuelles et répétitives qui occupent le temps sans jamais améliorer durablement la situation. Ce n'est pas simplement « du travail qu'on n'aime pas faire » : c'est une catégorie précise de tâches identifiable par des critères objectifs.

**Analogie fondatrice :** exécuter à la main `ssh serveur1 "systemctl restart app"` sur 50 serveurs chaque semaine est du toil. Écrire un playbook Ansible une fois pour automatiser ce redémarrage transforme une corvée récurrente en investissement permanent.

## Les six caractéristiques

Une tâche est probablement du travail ingrat si elle coche une ou plusieurs cases :

| Caractéristique | Ce que ça signifie | Exemple |
|---|---|---|
| Manuel | Intervention humaine directe | Se connecter en SSH pour supprimer des fichiers |
| Répétitif | Revient régulièrement | Redémarrer le même service chaque semaine |
| Automatisable | Un script pourrait le faire | Création de comptes via un simple formulaire |
| Tactique | Réaction à un problème, pas prévention | Répondre aux alertes « serveur down » |
| Sans valeur durable | Le problème reviendra | Fermer un ticket sans traiter la cause |
| Croissance linéaire | Plus d'infra = plus de travail | 10 serveurs = 10× plus de maintenance |

> [!info] Ce qui n'est PAS du travail ingrat
> Les réunions et la paperasse (overhead administratif, pas opérationnel) ; le travail fastidieux mais qui produit une amélioration permanente (nettoyer une config d'alertes mal faite) ; diagnostiquer un problème inédit (c'est de l'ingénierie).

## D'où vient-il

- **Tickets récurrents** : chaque ticket traité en génère de nouveaux (création de comptes, config de load balancers, règles firewall une par une) — le système « fonctionne » du point de vue utilisateur, donc personne ne questionne le processus.
- **Interruptions de production** : libérer un disque saturé, redémarrer une appli qui fuit la mémoire, remplacer un disque défaillant.
- **Déploiements manuels** : rollbacks, patchs d'urgence de nuit, changements de config répétés serveur par serveur.
- **Migrations qui s'éternisent** : bases de données, passage au cloud, changement d'outil de configuration.

## Mesurer avant d'agir

« J'ai l'impression de perdre du temps » n'est pas mesurable. Méthode en 3 étapes : inventorier les sources avec les personnes qui font le travail → choisir une unité (heures/semaine, nombre de tickets, interventions manuelles) → suivre dans le temps (avant/pendant/après).

Avant d'automatiser, calculer le ROI : temps pour automatiser (dev + maintenance) vs temps économisé (fréquence × durée) — sans oublier les bénéfices indirects (moins d'erreurs humaines, moral d'équipe, formation plus rapide via processus documenté).

## Sept stratégies pour réduire les corvées

1. **Éliminer à la source** — avant d'automatiser, chercher si le besoin peut disparaître (ex. corriger un driver réseau mal configuré plutôt qu'automatiser la purge des logs qu'il génère).
2. **Refuser certaines tâches** — analyser le coût réel de ne pas intervenir, grouper les demandes par lots.
3. **Utiliser les SLO comme filtre** — avec un budget d'erreur défini, une alerte mineure qui ne le menace pas peut attendre.
4. **Commencer par une interface humaine** — structurer les demandes (formulaire plutôt qu'email libre) avant d'automatiser les 20% de cas qui couvrent 80% des demandes.
5. **Proposer du self-service** — laisser l'utilisateur faire lui-même (formulaire web, script documenté) ce qu'il demandait auparavant.
6. **Commencer petit, itérer** — automatiser d'abord les 2-3 tâches les plus pénibles, mesurer, améliorer.
7. **Standardiser l'environnement** — principe *cattle not pets* : serveurs interchangeables, configuration cohérente (Infrastructure as Code), outils et procédures identiques partout.

## Bonnes pratiques d'automatisation et erreurs courantes

| À éviter | Conséquence | Solution |
|---|---|---|
| Automatiser sans documenter | Personne ne comprend, on devient indispensable (donc prisonnier) | Documenter en même temps qu'on automatise |
| Ignorer les corvées | Épuisement, démotivation, départs | Mesurer et rendre le problème visible |
| Automatiser à moitié | Double travail (script à maintenir + tâche manuelle en secours) | Aller jusqu'au bout ou ne pas commencer |
| Scripts sans versionnement | Dette technique ingérable, versions divergentes | Git dès le premier jour, tests, CI/CD |
| Pas de métriques | Impossible de justifier le temps investi | Mesurer le temps avant/après |
| Automatiser le critique en dernier | Les tâches à plus fort impact restent des points de défaillance humaine | Prioriser par impact |

Côté conception : valider les entrées même venant de systèmes de confiance, intégrer des garde-fous (timeouts, vérification d'état avant action), prévoir un repli vers un humain en cas de doute. Une automatisation qui couvre 80% des cas est déjà une victoire — ne pas bloquer sur les 20% marginaux. Préférer des briques réutilisables à des scripts monolithiques.

## Cas particuliers

> [!tip] La règle des 50% (Google)
> Chez Google SRE, pas plus de 50% du temps d'une équipe sur le travail opérationnel — le reste doit servir à l'ingénierie et à l'automatisation. Sans ce garde-fou, le toil tend à occuper 100% du temps : plus on en fait, moins on a de temps pour l'éliminer (cercle vicieux).

> [!warning] Le toil est un emprunt avec intérêts
> Ne pas automatiser une tâche fait gagner du temps dans l'instant, mais elle est remboursée avec intérêts à chaque répétition — le coût cumulé dépasse toujours celui de l'automatisation initiale.

## Checklists pratiques

### Pour commencer (minimum viable)

- [ ]  Lister les tâches récurrentes de l'équipe
- [ ]  Mesurer le temps passé sur les corvées (heures/semaine)
- [ ]  Identifier le top 3 des plus pénibles
- [ ]  Automatiser une première tâche critique
- [ ]  Documenter le processus automatisé
- [ ]  Versionner les scripts (Git)
- [ ]  Mettre en place des métriques de base
- [ ]  Planifier une revue trimestrielle

### Pour une organisation mature

- [ ]  Budget temps dédié à la réduction des corvées (ex. 20%)
- [ ]  Dashboard de suivi visible par tous
- [ ]  Self-service pour les demandes courantes
- [ ]  Indicateurs dans les objectifs d'équipe
- [ ]  Feedback continu des utilisateurs
- [ ]  Documentation centralisée et à jour
- [ ]  Procédures de rollback automatisées
- [ ]  Partage des victoires avec l'organisation

## Prérequis & suite

- [[Operations — Index des fiches]] ← retour à l'index du module
- [[Bash — Index des fiches]] ← l'automatisation concrète des tâches identifiées ici passe par des scripts
- [[Cron — Index des fiches]] ← l'analogie fondatrice (remplacer une commande manuelle répétée par une tâche planifiée) trouve sa suite naturelle ici
- [[Manques]] → Dette technique, Patch management, Baseline & Drift, Gestion des capacités (P4, non couverts) : suites logiques de ce module Operations
