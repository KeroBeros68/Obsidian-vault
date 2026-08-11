#devops #documentation #fondamentaux

## Le coût réel de l'absence de documentation

L'absence de documentation n'est pas neutre : elle génère un coût mesurable, souvent invisible car diffus dans le temps de travail quotidien.

| Constat | Donnée | Source citée |
|---------|--------|----------------|
| Temps passé à chercher de l'information | ~2,5h/jour, soit 30 % du temps d'un travailleur du savoir | IDC |
| Productivité perdue par friction informationnelle | 25 % | APQC |
| Coût équivalent pour une équipe de 10 personnes | ~3 postes à temps plein perdus en recherche d'information | — |
| MTTR (temps moyen de résolution d'incident) sans documentation | 4 à 8 heures | — |
| MTTR avec documentation (runbook) | ~20 minutes (10 à 20x plus rapide) | — |
| Onboarding sans documentation | 3 à 6 mois pour devenir autonome | — |
| Onboarding avec documentation | 3 à 6 semaines | — |

> [!info] Fiabilité des chiffres
> Ces données sont celles citées par l'article source (IDC, APQC, Forrester, Atlassian *State of Incident Management*, Google SRE Book) ; elles n'ont pas été vérifiées indépendamment auprès des études primaires. À prendre comme ordre de grandeur illustrant une tendance forte plutôt que comme mesure exacte applicable à toute organisation.

## Le bus factor

Le **bus factor** (ou *truck factor*, parfois *lottery factor*) mesure le nombre de personnes qui peuvent disparaître d'une équipe (départ, maladie, vacances) avant que le fonctionnement du système ne soit bloqué faute de connaissance.

- Bus factor = 1 → risque maximal : une seule personne détient une information critique.
- Bus factor ≥ 3 → résilience assurée : la connaissance est répartie et accessible sans dépendre d'un individu.

### Connaissance tacite vs explicite

- **Connaissance tacite** : savoir non formalisé, dans la tête d'une personne (raccourcis appris par l'expérience, pièges connus par habitude).
- **Connaissance explicite** : savoir écrit, partagé, consultable sans dépendre de la personne qui l'a acquis.

Le rôle de la documentation est précisément de transformer la connaissance tacite en connaissance explicite, avant qu'elle ne disparaisse avec la personne qui la détient.

## Le principe fondamental

> [!tip] Règle mémo
> Si une information critique n'existe que dans la tête d'une seule personne, elle n'existe pas — au sens où l'organisation ne peut pas compter dessus de façon fiable.

Documenter n'est donc pas une tâche annexe de "confort" mais un mécanisme de **réduction de risque opérationnel**, au même titre qu'un test automatisé ou une sauvegarde. L'article source la compare à une assurance habitation : on ne documente pas parce qu'on prédit un incident précis, mais par prudence face à un risque probable sur la durée.

## La documentation comme protection, sur trois plans

| Domaine protégé | Mécanisme | Exemple |
|-------------------|-----------|---------|
| **Incidents** | Un runbook permet à n'importe qui de résoudre un incident connu, pas seulement l'expert | Un runbook "connexions PostgreSQL saturées" cité comme utilisé 14 fois en un an, ~12 min de résolution en moyenne |
| **Turnover** | Un service overview (2-3 pages) transmet le contexte nécessaire à la succession d'une personne qui part | Fonctionnalité, dépendances, déploiement, surveillance, raisons des choix d'architecture |
| **Scaling** | Au-delà de 3 personnes, la communication informelle (oral, couloir) ne suffit plus à propager l'information de façon fiable | La documentation devient le "système nerveux" de l'équipe |

## Illustration — le minimum vital à documenter

Face à un système non documenté, prioriser plutôt que viser l'exhaustivité :

```
1. Runbooks pour les 3 incidents les plus fréquents/impactants   ← meilleur retour sur effort
2. 1 checklist de déploiement en production
3. 1 service overview (2-3 pages) par service critique
4. Des ADRs pour les décisions d'architecture structurantes
```

Voir [[Documentation 05 — Outils et maturité (Docs-as-Code vs Wiki)]] pour la trajectoire complète de mise en place (Urgence → Fondations → Maturité).

## Cas particuliers

> [!warning] Le cercle vicieux du "on documentera plus tard"
> Équipe débordée → pas le temps de documenter → expertise concentrée sur peu de personnes → incidents plus longs à résoudre sans documentation → équipe encore plus débordée → retour au point de départ. Le "plus tard" ne survient jamais spontanément ; le contexte d'une décision ou d'un incident se perd généralement après quelques mois s'il n'est pas noté à chaud.

> [!warning] Documenter n'élimine pas le risque à lui seul
> Une documentation qui existe mais n'est jamais mise à jour ou jamais relue donne une **fausse impression de sécurité** : le bus factor réel reste bas si personne ne sait que le document existe ou s'il est obsolète. Voir [[Documentation 02 — Gouvernance et Definition of Done]] pour le mécanisme qui maintient la documentation vivante.

> [!info] Ce n'est pas propre au DevOps
> Le principe du bus factor s'applique à toute connaissance critique (code, infrastructure, processus métier), mais il est particulièrement visible en exploitation (SRE, ops) où une indisponibilité a un coût direct et mesurable.
