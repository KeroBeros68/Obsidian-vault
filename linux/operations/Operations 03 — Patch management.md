#linux #operations #sre #sécurité #patch-management

## Qu'est-ce que le patch management

Le **patch management** ne se limite pas à lancer `apt upgrade`. C'est un processus complet, de la veille sur les vulnérabilités jusqu'à la vérification post-déploiement.

| Composant | Ce qu'il couvre |
|---|---|
| Veille | Suivre les annonces CVE, bulletins éditeurs, advisories |
| Inventaire | Savoir ce qui tourne, où, dans quelle version |
| Évaluation | Prioriser selon la criticité et l'exposition |
| Test | Valider que le patch ne casse rien |
| Déploiement | Appliquer de façon contrôlée |
| Vérification | S'assurer que tout fonctionne après |
| Documentation | Garder une trace pour l'audit |

> [!tip] Analogie médicale
> Un patch, c'est comme un médicament : vérifier que le patient en a besoin (évaluation), tester qu'il n'y a pas d'allergie (staging), adapter la posologie (déploiement progressif), surveiller les effets secondaires (vérification). Un admin qui patche sans évaluer est aussi dangereux qu'un médecin qui prescrit sans examen.

## Les types de correctifs

| Type | Urgence | Exemples |
|---|---|---|
| Sécurité critique | Immédiate | CVE avec exploit public, CVSS ≥ 9.0 |
| Sécurité haute | Sous 7 jours | CVE sans exploit, CVSS 7.0-8.9 |
| Sécurité moyenne | Sous 30 jours | CVE difficile à exploiter, CVSS 4.0-6.9 |
| Fonctionnel | Planifié | Bug fixes, améliorations |
| Performance | Planifié | Optimisations |

## Le dilemme du patch management

Patcher vite expose à des régressions ; patcher tard expose à des compromissions.

| Patcher trop vite | Patcher trop tard |
|---|---|
| Régression applicative | Vulnérabilité exploitée |
| Downtime non planifié | Compromission système |
| Perte de confiance utilisateurs | Perte de données |
| Stress de l'équipe | Amende RGPD/conformité |

> [!warning] Les chiffres qui font mal
> 60% des breaches impliquent une vulnérabilité connue pour laquelle un patch existait déjà (Verizon DBIR). 12 jours : délai médian entre la publication d'un exploit et sa première utilisation en attaque. 103 jours : délai moyen pour patcher une vulnérabilité critique en grande entreprise. Chaque jour sans patch est un jour de risque.

## Le cycle de gestion des patchs (7 étapes)

### 1. Surveiller — la veille continue

Bulletins éditeurs (RHSA pour RHEL/CentOS, USN pour Ubuntu, DSA pour Debian, SUSE Security Advisories), bases de référence CVE/NVD, scores CVSS. Outils de scan automatisé : OpenSCAP, Trivy, Lynis (open source), Nessus, Qualys (commercial).

```bash
# Vérifier les CVE sur les packages installés
yum updateinfo list security                          # RHEL/CentOS
apt list --upgradable 2>/dev/null | grep -i security   # Ubuntu/Debian

# Scanner un serveur avec Trivy
trivy rootfs --severity HIGH,CRITICAL /
```

### 2. Évaluer — prioriser intelligemment

| Critère | Question à poser |
|---|---|
| CVSS | Quel est le score ? ≥ 9.0 = critique |
| Exposition | Le service est-il accessible depuis Internet ? |
| Exploit | Un exploit public existe-t-il ? |
| Impact | Que se passe-t-il si la vulnérabilité est exploitée ? |
| Compensations | Y a-t-il déjà des mesures d'atténuation ? |

> [!example] Exemple de priorisation
> CVE OpenSSL RCE, CVSS 9.8 (critique), exposition serveurs web publics (haute), exploit POC public disponible (urgence), impact exécution de code arbitraire → décision : patch sous 24h.

### 3. Tester — valider avant de déployer

Ne jamais patcher en production sans staging représentatif (même OS, mêmes versions, mêmes configurations).

```bash
# Snapshot avant patch (si VM)
virsh snapshot-create-as serveur-staging pre-patch

# Appliquer le patch
apt update && apt upgrade -y paquet-concerné
needrestart -r a   # redémarrer les services si nécessaire
```

Valider ensuite : l'application démarre-t-elle ? les services répondent-ils ? les tests automatisés passent-ils ? les performances sont-elles stables ? Laisser tourner quelques heures ou jours avant de valider pour la production.

### 4. Planifier — communiquer et coordonner

Un patch non communiqué est un risque d'incident.

| Élément | Contenu |
|---|---|
| Fenêtre | Date, heure de début, durée estimée |
| Périmètre | Serveurs concernés, services impactés |
| Risques | Possibilité de downtime, rollback prévu |
| Responsables | Qui déploie, qui valide, qui est de garde |
| Communication | Qui prévenir, par quel canal |

### 5. Déployer — de façon contrôlée

| Stratégie | Fonctionnement | Avantage |
|---|---|---|
| Canary | Patcher un petit groupe (10%), observer, élargir progressivement | Si problème, seul un petit groupe est impacté |
| Rolling update | Patcher serveur par serveur, en le sortant/remettant du load balancer | Zéro downtime si bien orchestré |
| Blue-Green | Préparer un environnement complet patché (Green), basculer le trafic depuis Blue | Rollback instantané (rebascule vers Blue) |

```bash
# Canary avec Ansible
ansible-playbook patch.yml -l canary_group   # 10% des serveurs
# observation 2h...
ansible-playbook patch.yml -l group_a        # 50%
# observation 4h...
ansible-playbook patch.yml -l group_b        # reste
```

### 6. Vérifier — s'assurer que tout fonctionne

```bash
dpkg -l | grep paquet-concerné                    # ou rpm -qa | grep paquet-concerné
systemctl status service-critique
journalctl -p err -n 50 --no-pager
curl -I https://mon-service.example.com/health
```

Checklist post-déploiement :

- [ ]  Version du package correcte
- [ ]  Services démarrés et healthy
- [ ]  Pas d'erreurs dans les logs
- [ ]  Monitoring vert
- [ ]  Tests applicatifs OK
- [ ]  Performance stable

### 7. Documenter — pour l'audit et l'historique

| Information | Exemple |
|---|---|
| Date/heure | 2025-02-15 02:15 UTC |
| CVE/Advisory | CVE-2024-XXXX, RHSA-2025:0123 |
| Serveurs | web-prod-01 à web-prod-10 |
| Version avant/après | openssl-1.1.1k-1 → 1.1.1k-2 |
| Responsable | alice@ops |
| Résultat | Succès, aucun rollback |

Sert pour les audits de conformité (PCI-DSS, ISO 27001), l'analyse post-incident et la planification future.

## Automatisation

| Outil | Environnement | Capacités |
|---|---|---|
| Ansible | Tout Linux | Playbooks de patch, rolling updates |
| Puppet | Tout Linux | Gestion de configuration + patch |
| Satellite | RHEL | Gestion complète du cycle de vie |
| Landscape | Ubuntu | Gestion centralisée des patchs |
| Spacewalk | Multi-distro | Alternative open source à Satellite |
| Rudder | Tout Linux | Interface web sans code, agent qui corrige les écarts toutes les 5 min, rapports de conformité (CIS, PCI-DSS, ANSSI) |

Un playbook Ansible de patch typique : snapshot pré-patch → retrait du load balancer → mise à jour des packages → redémarrage si nécessaire → vérification des services critiques et d'un healthcheck applicatif → remise dans le load balancer.

### Mises à jour automatiques : oui ou non ?

| Automatique sans validation | Automatique avec validation |
|---|---|
| ❌ Production critique | ✅ Environnements de dev |
| ❌ Applications sensibles | ✅ Serveurs non critiques |
| ❌ Systèmes réglementés | ✅ Avec rollback automatique |

```
# /etc/apt/apt.conf.d/50unattended-upgrades (Ubuntu)
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";  // pas les mises à jour standards
};
Unattended-Upgrade::Package-Blacklist {
    "linux-";   // ⚠️ pas de mise à jour kernel automatique
    "mysql-";   // ⚠️ pas de mise à jour BDD automatique
};
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";
```

## Le cas particulier des systèmes legacy

Certains systèmes ne peuvent pas être patchés (fin de support, contraintes métier) :

1. **Isoler** — VLAN dédié, règles firewall strictes, flux limités au strict nécessaire.
2. **Durcir** — désactiver les services inutiles, configuration restrictive.
3. **Surveiller** — monitoring renforcé (HIDS), analyse des logs, alertes sur comportements anormaux.
4. **Planifier le remplacement** — un système legacy est une bombe à retardement ; documenter le plan de migration et obtenir un budget.
5. **Accepter le risque formellement** — si le système reste, faire valider le risque par la direction, par écrit.

## Métriques (KPIs)

| Métrique | Ce qu'elle mesure | Objectif |
|---|---|---|
| MTTP (Mean Time To Patch) | Délai entre publication CVE et patch appliqué | Critique < 72h, Haute < 7j |
| Taux de couverture | % de systèmes à jour | > 95% |
| Patchs en retard | Nombre de patchs non appliqués dans les délais | 0 |
| Taux de rollback | % de patchs nécessitant un retour arrière | < 5% |
| Incidents liés au patch | Nombre d'incidents causés par un patch | 0 idéalement |

## Pièges classiques

> [!warning] Patcher en prod sans tester
> Risque de régression directe. Toujours tester en staging d'abord.

> [!warning] Ignorer les patchs « pas critiques »
> Ils s'accumulent silencieusement. Prévoir des fenêtres régulières pour tous les niveaux de criticité.

> [!warning] Pas de rollback prévu
> On reste coincé en cas de problème. Snapshot systématique ou stratégie blue-green.

> [!warning] Patcher le vendredi soir
> Personne n'est disponible pour réagir en cas d'incident. Patcher en début de semaine.

> [!warning] Tout patcher d'un coup
> Blast radius maximal si un patch casse quelque chose. Préférer un déploiement progressif (canary).

> [!warning] Blâmer en cas d'incident
> Pousse l'équipe à cacher les problèmes plutôt qu'à les signaler. Adopter une culture blameless.

## Checklists pratiques

### Avant de patcher

- [ ]  Inventaire vérifié — je sais exactement quels serveurs sont concernés
- [ ]  Criticité évaluée — CVSS, exposition, existence d'exploits analysés
- [ ]  Test réalisé — patch validé en staging sans régression
- [ ]  Rollback préparé — snapshot créé, procédure documentée et testée
- [ ]  Fenêtre planifiée — date communiquée, équipes informées, astreinte prévenue
- [ ]  Monitoring prêt — dashboards ouverts, alertes configurées

### Après le patch

- [ ]  Patch appliqué confirmé — version vérifiée sur tous les serveurs concernés
- [ ]  Services fonctionnels — tous les services critiques démarrés et sains
- [ ]  Tests passés — automatisés et manuels validés
- [ ]  Monitoring vert — pas d'alerte, métriques stables
- [ ]  Documentation mise à jour — ticket fermé, CMDB à jour, rapport enregistré
- [ ]  Période d'observation — surveillance renforcée pendant 24-48h

## Prérequis & suite

- [[Operations — Index des fiches]] ← retour à l'index du module
- [[Operations 01 — Réduire le travail ingrat]] ← l'automatisation du patching (veille, déploiement, vérification) est une application directe de la réduction du toil
- [[Operations 02 — Gérer la dette technique]] ← un legacy non patché est une forme de dette d'infrastructure
- [[Manques]] → Baseline & Drift, Gestion des capacités (P4, non couverts) : suites logiques de ce module Operations
