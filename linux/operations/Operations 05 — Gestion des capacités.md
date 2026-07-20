#linux #operations #sre #capacity-planning

## Qu'est-ce que la planification des capacités

Le **capacity planning** consiste à mesurer ce qui est utilisé, observer la tendance, et prévoir quand ajouter des ressources — avant d'atteindre la saturation, sans pour autant surdimensionner inutilement.

> [!tip] La métaphore du réservoir
> Chaque ressource (CPU, RAM, disque) est un réservoir d'eau : niveau actuel (combien est utilisé), débit d'entrée (à quelle vitesse ça se remplit), capacité totale (quand sera-t-il plein). Le capacity planning répond à une question simple : dans combien de temps le réservoir débordera-t-il ?

Sans capacity planning, on subit les événements (disque plein découvert à 3h du matin, provisionnement dans la panique, surdimensionnement « au cas où » qui gaspille le budget, ou sous-dimensionnement qui fait ramer le site pendant les pics). Avec, on anticipe : mesurer régulièrement, calculer le runway, définir des seuils, prévoir la croissance.

## Le concept clé : le runway

Le **runway** (« piste d'atterrissage ») est le temps restant avant qu'une ressource soit saturée, au rythme actuel de croissance :

```
Runway = (Capacité totale − Utilisation actuelle) / Taux de croissance
```

> [!example] Exemple
> Disque de 1 To, utilisé à 700 Go, qui grossit de 50 Go/mois → capacité restante 300 Go / 50 Go/mois = **runway de 6 mois**. Il faut planifier l'extension maintenant, pas dans 5 mois.

Le runway transforme une métrique technique (« 700 Go utilisés ») en information actionnable (« il reste 6 mois ») : il permet de planifier le budget, de prioriser (2 semaines de runway est plus urgent que 2 ans), et de communiquer clairement (« commander avant mars » plutôt que « le disque est à 70% »).

> [!warning] Attention aux pics
> Le runway est calculé sur la croissance moyenne. Un pic (Black Friday, campagne marketing) peut le raccourcir brutalement. Toujours ajouter une marge de sécurité de 20-30%.

## Les quatre ressources à planifier

| Ressource | Spécificité | Métriques clés | Runway |
|---|---|---|---|
| CPU | La plus « élastique » — en manquer ralentit sans arrêter le service | Utilisation moyenne (`mpstat`), P95, load average (`uptime`) | Moins critique, l'auto-scaling absorbe souvent le problème ; surveiller les pics (P95 > 80%) |
| RAM | Plus critique — pleine, elle déclenche le swap (lent) ou l'OOM-Killer | Mémoire disponible (`free -h`), swap utilisé, tendance | Critique si le swap est actif ; runway < 1 mois = action immédiate |
| Disque | La plus prévisible (croissance linéaire) mais la plus dangereuse (disque plein = panne totale) | Espace utilisé (`df -h`), croissance quotidienne, inodes (`df -i`) | Calculable précisément — souvent la ressource la plus facile à prévoir |
| Réseau | Rarement le goulot d'étranglement, mais brutal quand il l'est | Bande passante (`iftop`), connexions actives (`ss -s`), erreurs (`ip -s link`) | Difficile à prévoir, très variable ; surveiller pics et erreurs |

> [!warning] Le piège des inodes
> Un disque peut avoir de l'espace libre mais être saturé en inodes (trop de fichiers). Surveiller les deux métriques avec `df -h` **et** `df -i`.

## Définir les bons seuils d'alerte

Deux niveaux d'alerte par ressource : **warning** (on approche de la limite → planifier) et **critical** (on est proche de la saturation → agir immédiatement).

| Ressource | Seuil d'alerte | Seuil critique | Pourquoi |
|---|---|---|---|
| CPU | > 70% (P95) | > 85% | Le CPU absorbe des pics courts |
| RAM | > 75% | > 85% | Laisser de la marge avant le swap |
| Disque | > 75% | > 85% | Les écritures ralentissent quand c'est plein |
| Connexions | > 60% max | > 80% max | Les nouvelles connexions seront refusées |

Ces valeurs sont des points de départ — ajuster selon la criticité du serveur, la réactivité de provisionnement (cloud = seuils plus hauts possibles) et l'historique des pics réels.

## Méthode pas à pas

1. **Établir une baseline** — mesurer l'état actuel de chaque ressource et noter la date de référence :
   ```bash
   mpstat 1 10 | tail -1   # CPU : utilisation moyenne
   free -h                 # RAM : mémoire disponible
   df -h                   # Disque : espace utilisé
   ss -s                   # Réseau : connexions actives
   ```
2. **Collecter l'historique** — au moins 30 jours de données pour calculer une tendance fiable. Avec Prometheus : `rate(node_filesystem_size_bytes{mountpoint="/"}[30d])`. Sans outil, noter les valeurs chaque semaine pendant un mois.
3. **Calculer le runway** pour chaque ressource critique et prioriser (ex. : disque `/data` à 2 To, utilisé 1,8 To, +100 Go/mois → runway 2 mois, action urgente).
4. **Configurer les alertes** — ex. dans Prometheus/Alertmanager, une règle `warning` sous 75% d'espace disponible et `critical` sous 85%.
5. **Planifier les actions** — cloud : upgrade/auto-scaling ; on-premise : commander le matériel (tenir compte du délai de livraison) ; obtenir la validation budgétaire.
6. **Réviser mensuellement** — les prévisions étaient-elles justes ? y a-t-il eu des imprévus ? faut-il ajuster le modèle ? Documenter les écarts pour affiner les prochaines prévisions.

## Prévoir la croissance future

Le runway dit où on en est ; le **forecasting** dit où on va.

| Méthode | Principe | Quand l'utiliser | Limite |
|---|---|---|---|
| Tendance linéaire | La croissance future ressemble à la croissance passée : `Utilisation dans N mois = Utilisation actuelle + (Croissance mensuelle × N)` | Croissance régulière, pas de changement majeur prévu | Ne capture pas les accélérations/ralentissements |
| Saisonnalité | Certaines périodes sont plus chargées (Noël, soldes, fin de mois) : comparer les mêmes périodes d'une année sur l'autre | Patterns répétitifs observables dans les données | Nécessite au moins 1 an d'historique |
| Scénarios | Définir plusieurs futurs possibles (optimiste +10%/mois, réaliste +20%/mois, pessimiste +40%/mois) | Incertitude élevée, lancement de nouveauté | Dimensionner sur le scénario pessimiste pour les lancements |

> [!tip] Règle des 20-30%
> Toujours ajouter une marge de sécurité aux prévisions — les pics imprévus arrivent toujours au pire moment. Ex. : prévision à 800 Go dans 6 mois + marge 25% = dimensionner pour 1 To.

## Le cycle du capacity planning

Ce n'est pas un exercice ponctuel mais un cycle continu :

| Activité | Fréquence | Qui |
|---|---|---|
| Vérifier les alertes | Quotidien | Ops/SRE |
| Mettre à jour les dashboards | Hebdomadaire | Ops/SRE |
| Calculer les runways | Mensuel | Ops/SRE + Tech Lead |
| Réviser les prévisions | Trimestriel | Équipe + Management |
| Planifier le budget | Annuel | Management + Finance |

## Tester la capacité réelle : le load testing

Calculer le runway est utile, mais tester la capacité réelle confirme les prévisions.

| Type de test | Objectif | Quand l'utiliser |
|---|---|---|
| Load test | Vérifier le comportement sous charge normale | Régulièrement (mensuel) |
| Stress test | Trouver le point de rupture | Avant les événements majeurs |
| Spike test | Tester la réaction aux pics soudains | Après des changements d'architecture |

```bash
# Test simple avec hey (ou ab / Apache Bench)
hey -n 1000 -c 50 https://votre-site.com/
```

## Pièges classiques

> [!warning] Pas de mesures
> On ne sait pas où on en est. Installer un monitoring de base avant toute chose.

> [!warning] Marge zéro
> Le moindre pic cause une panne. Toujours 20-30% de marge sur les prévisions.

> [!warning] Oublier les pics
> Dimensionner sur la moyenne masque les pics réels. Utiliser le P95, pas la moyenne.

> [!warning] Cloud = infini
> L'auto-scaling a des limites (quotas, coûts), met 5-10 minutes à réagir, et certaines ressources ne scalent pas (disque, base de données). Le capacity planning reste nécessaire même avec l'auto-scaling.

> [!warning] Provisionnement tardif
> Commander quand c'est déjà plein est trop tard. Agir au seuil d'alerte (warning), pas seulement au seuil critique.

## Checklists pratiques

### Niveau 1 — Minimum viable (1-2 jours de travail)

- [ ]  Métriques CPU, RAM, disque collectées
- [ ]  Dashboard de capacity visible
- [ ]  Alertes sur les seuils critiques (disque > 85%, RAM > 85%)
- [ ]  Runway calculé pour le disque principal
- [ ]  Revue mensuelle planifiée dans l'agenda

### Niveau 2 — Mature (1-2 semaines)

- [ ]  Historique des métriques conservé (3+ mois)
- [ ]  Runway calculé pour toutes les ressources critiques
- [ ]  Modèle de prévision formalisé (même simple)
- [ ]  Budget infra connu et suivi
- [ ]  Communication régulière avec le métier
- [ ]  Load testing annuel

### Niveau 3 — Avancé (effort continu)

- [ ]  Prévisions par service/application
- [ ]  Scénarios documentés (optimiste/réaliste/pessimiste)
- [ ]  Automatisation des rapports
- [ ]  Intégration avec le processus budgétaire
- [ ]  Simulation de pics avant événements majeurs
- [ ]  Post-mortem après chaque pic (Black Friday, etc.)

## Prérequis & suite

- [[Operations — Index des fiches]] ← retour à l'index du module
- [[Operations 01 — Réduire le travail ingrat]] ← automatiser la collecte de métriques et les alertes de capacité réduit le toil de surveillance manuelle
- [[Operations 03 — Patch management]] ← un runway court peut coïncider avec une fenêtre de patch, à coordonner
- [[Operations 04 — Baseline & Drift]] ← une dérive de configuration (ex. service mal configuré consommant plus de ressources) peut fausser les prévisions de capacité
- [[Manques]] → cette fiche clôt la série SRE `linux/operations/` planifiée depuis Operations 01
