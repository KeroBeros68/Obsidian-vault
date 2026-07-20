#linux #operations #sre #sécurité #drift

## Qu'est-ce que la dérive de configuration

L'**état de référence** (*baseline*) est l'état validé et documenté d'un système — la définition de « comment ce serveur devrait être configuré ». Il peut être documenté (wiki, CMDB), codifié (playbooks Ansible, manifests Terraform, Dockerfiles) ou implicite (« c'est comme ça qu'on fait » — dangereux).

| Composant | Exemples d'éléments contrôlés |
|---|---|
| Système | Version OS, paramètres noyau, utilisateurs |
| Packages | Liste des logiciels installés, versions |
| Configuration | Fichiers de conf, permissions, services actifs |
| Réseau | Interfaces, routes, firewall, DNS |
| Sécurité | Durcissement, certificats, politiques |

La **dérive de configuration** (*configuration drift*) survient quand l'état réel d'un système s'écarte de son état de référence : `État attendu − État réel = Dérive`.

> [!tip] Analogie Git
> L'état de référence est la branche `main`. La dérive arrive quand chaque développeur modifie sa branche `feature` sans jamais merge/rebase. Après quelques mois, plus personne ne peut merger sans conflits massifs — chaque branche a divergé dans sa direction. Un workflow qui impose des merges réguliers (l'équivalent d'un scan de drift) évite la divergence.

> [!info] Ce qui n'est PAS de la dérive
> Un changement planifié et documenté (évolution contrôlée de l'état de référence) ; une mise à jour de sécurité appliquée uniformément ; une différence volontaire entre environnements (dev ≠ prod par conception) ; un bug logiciel (défaut, pas un écart de configuration).

## Pourquoi la dérive est dangereuse

| Conséquence | Ce qui se passe | Exemple réel |
|---|---|---|
| Incidents | Comportement imprévisible | Un serveur différent des autres plante lors d'une mise à jour |
| Failles de sécurité | Durcissement perdu | Twilio (2020) : fuite de données via un bucket S3 dont la config avait dérivé |
| Non-conformité | Écart aux politiques | Audit PCI-DSS échoué car des serveurs n'avaient plus les bons paramètres |
| Temps perdu | Debug de « snowflakes » | Chaque serveur différent = chaque incident unique à investiguer |

> [!warning] La spirale de la dette de configuration
> Comme une boule de neige qui dévale une pente : **dérive initiale** (« je corrige vite en SSH », quelques minutes de coût) → **accumulation** (modifications non documentées qui s'empilent, quelques heures) → **divergence** (« ça marche chez moi mais pas en prod », jours de travail) → **snowflake** (serveur unique, personne n'ose plus y toucher, semaines) → **paralysie** (toute mise à jour devient un projet risqué, mois de refonte souvent après un incident majeur).

Un serveur **snowflake** a subi tant de modifications manuelles non documentées que plus personne ne sait comment le reproduire — s'il tombe, sa reconstruction prend des jours voire des semaines.

## Sources et types de dérive

**Dérive technique** (causes opérationnelles, souvent faciles à résoudre par l'outillage) :

| Source | Comment ça arrive | Pourquoi c'est dangereux |
|---|---|---|
| Modifications manuelles | SSH + commandes ad hoc pour « corriger vite » | Pas de trace, pas de rollback |
| Patchs et mises à jour | Correctif appliqué sur certains serveurs seulement | Parc hétérogène |
| Automatisations partielles | Playbook qui échoue à mi-chemin | État incohérent |
| Ressources non gérées | Création cloud via console au lieu d'IaC | Shadow IT invisible |

**Dérive organisationnelle** (causes humaines, nécessitent des changements culturels) :

| Source | Comment ça arrive | Pourquoi c'est dangereux |
|---|---|---|
| Processus contournés | « Pas le temps pour le change management » | Modifications non tracées |
| Documentation obsolète | L'état de référence « officiel » date d'il y a des années | On ne sait plus ce qui est correct |
| Silos équipes | Dev, Ops, Sécu travaillent séparément | Chacun modifie de son côté |
| Turnover | L'expert qui savait tout est parti | Connaissances perdues |

## Le cycle de gestion de la dérive

Comme la [[Operations 02 — Gérer la dette technique|dette technique]], c'est un processus continu : détecter, analyser, corriger, prévenir.

### 1. Détecter — comparer l'état réel à l'état de référence

```bash
# Terraform : voir les différences entre état déclaré et réel
terraform plan

# Ansible : mode check (dry-run)
ansible-playbook site.yml --check --diff

# Puppet : mode noop
puppet agent --test --noop

# OpenSCAP : audit contre un état de référence CIS
oscap xccdf eval --profile cis --results results.xml /usr/share/xml/scap/ssg-rhel8-ds.xml

# Lynis : audit de sécurité
lynis audit system
```

Limite des outils IaC : ils ne voient que ce qu'ils gèrent — une modification faite en dehors de l'outil reste invisible. D'où l'intérêt d'outils de scan dédiés (AIDE, Tripwire, driftctl, Snyk IaC, Spacelift) qui détectent aussi les ressources « orphelines ».

### 2. Analyser — comprendre avant de corriger

Ne jamais corriger une dérive à l'aveugle : elle peut révéler un problème plus profond, ou l'état de référence lui-même peut être celui à faire évoluer.

| Question à poser | Pourquoi c'est important |
|---|---|
| Qui a fait la modification ? | Identifier un problème de processus ou de compétence |
| Quand a-t-elle eu lieu ? | Corréler avec des incidents ou des déploiements |
| Pourquoi était-elle nécessaire ? | Peut révéler un besoin que l'état de référence ne couvre pas |
| Quel impact si on corrige ? | Éviter de casser quelque chose en « réparant » |

> [!example] Exemple
> Un serveur a une règle firewall non standard. Avant de la supprimer, vérifier qu'elle n'a pas été ajoutée pour bloquer une attaque en cours — si c'est le cas, il faut l'intégrer à l'état de référence, pas la supprimer.

### 3. Corriger — rétablir ou faire évoluer

| Option | Quand l'utiliser | Comment |
|---|---|---|
| Rétablir l'état de référence | La dérive était une erreur | Réappliquer le playbook, `terraform apply` |
| Mettre à jour l'état de référence | La dérive était une amélioration | Modifier l'IaC puis redéployer |

```bash
# ❌ Mauvais : correction manuelle
sudo systemctl enable nginx

# ✅ Bon : correction via l'outil (Ansible ici)
ansible-playbook -l serveur23 site.yml --tags nginx
```

Corriger à la main crée une nouvelle dérive — un cercle vicieux.

### 4. Prévenir — empêcher la récidive

Automatiser tout changement (IaC, playbook, versionné — jamais de SSH pour « corriger vite ») ; détecter en continu (scan quotidien, pas seulement avant déploiement) ; bloquer les modifications manuelles (accès SSH restreints, policies IAM) ; documenter et former ; tracer et auditer (logs d'accès, audit trail cloud).

## Fix Forward ou Fix Backward ?

| Approche | Principe | Quand l'utiliser |
|---|---|---|
| Fix Backward | Rétablir l'état précédent (rollback) | La dérive a causé un problème |
| Fix Forward | Intégrer la modification à l'état de référence | La dérive était une amélioration |

> [!warning] Ne jamais corriger manuellement
> Quelle que soit l'approche choisie, se connecter au serveur pour « arranger les choses » crée une nouvelle dérive. Toujours passer par l'outil de gestion de configuration.

## Outils de détection par environnement

| Environnement | Outils | Points forts |
|---|---|---|
| Infrastructure cloud (IaC) | `terraform plan`, driftctl, Snyk IaC, Spacelift, Firefly | driftctl : open source, détecte aussi les ressources non managées (mais projet archivé, intégré à Snyk) |
| Serveurs Linux | AIDE, Tripwire, OpenSCAP, Lynis, OSSEC, Rudder | AIDE/Tripwire : changements de fichiers ; OpenSCAP : conformité CIS/SCAP ; Rudder : gestion + drift avec reporting |
| Orchestration | Ansible (`--check --diff`), Puppet (agent auto-correctif), Chef InSpec, SaltStack (`test=True`) | Détection intégrée au cycle de déploiement |

## Mesurer la dérive

| Métrique | Ce qu'elle mesure | Objectif |
|---|---|---|
| Taux de conformité | % de ressources sans dérive | > 95% |
| MTTD (Mean Time To Detect) | Temps entre dérive et détection | < 24h |
| MTTR (Mean Time To Remediate) | Temps entre détection et correction | < 48h |
| Ressources non gérées | Nombre de ressources hors IaC | Tendre vers 0 |
| Récurrence | Dérives qui reviennent après correction | 0 — sinon la cause racine n'est pas traitée |

Un dashboard de suivi (Grafana, Datadog, ou interne) doit répondre à : quelle est ma situation globale ? où sont les problèmes ? quelle est la tendance ? quelle est l'urgence ?

## Pièges classiques

> [!warning] Modifications manuelles non documentées
> Dérive invisible, pas de rollback possible. Tout faire passer par l'IaC.

> [!warning] État de référence « implicite »
> Personne ne sait ce qui est correct. Documenter et versionner l'état attendu.

> [!warning] Détection uniquement avant déploiement
> La dérive s'accumule entre-temps. Mettre en place un scan quotidien automatisé.

> [!warning] Correction sans analyse
> Risque de casser ce qui marchait. Toujours comprendre la cause avant d'agir.

> [!warning] Ignorer les alertes de drift
> « C'est normal, on verra plus tard » laisse la dette de configuration s'accumuler. Fixer un SLA de correction (ex. 48h).

## Checklists pratiques

### Minimum viable

- [ ]  État de référence documenté — au minimum un fichier décrivant la configuration attendue des serveurs critiques, idéalement du code IaC
- [ ]  Détection hebdomadaire — `terraform plan` ou audit AIDE/OpenSCAP au moins une fois par semaine
- [ ]  Processus de correction — savoir qui est responsable de corriger et sous quel délai
- [ ]  Historique des écarts — conserver une trace de chaque dérive détectée et de sa correction
- [ ]  Formation de base — l'équipe sait pourquoi les modifications manuelles sont interdites

### Organisation mature

- [ ]  IaC à 100% — toute l'infrastructure est codifiée, aucune ressource manuelle
- [ ]  Détection continue — scan de drift automatisé quotidien, alertes en temps réel
- [ ]  Blocage des modifications manuelles — policies IAM, accès SSH restreints, audit trail
- [ ]  SLA de correction — critique 4h, haute 24h, moyenne 48h, basse 1 semaine
- [ ]  Métriques et reporting — dashboard de conformité, rapports hebdomadaires au management
- [ ]  Revue post-incident — chaque dérive ayant causé un incident fait l'objet d'un postmortem
- [ ]  Amélioration continue — rétrospectives régulières sur la gestion de la dérive

## Prérequis & suite

- [[Operations — Index des fiches]] ← retour à l'index du module
- [[Operations 02 — Gérer la dette technique]] ← la dérive suit la même dynamique cumulative que la dette technique (coût croissant avec le temps)
- [[Operations 03 — Patch management]] ← un patch appliqué de façon incohérente sur le parc est une source directe de dérive
- [[Manques]] → Gestion des capacités (P4, non couverte) : dernière suite logique de ce module Operations
