#devops #documentation #pièges #erreurs #gouvernance

## 🪤 Piège 1 — Documentation sans owner nommé

```
❌ "L'équipe" maintient ce document
✅ Jean (ou le rôle "on-call lead") est owner de ce document
```

> [!warning] Une responsabilité collective n'est pas une responsabilité
> Sans owner nommé, chacun suppose que quelqu'un d'autre s'en occupe. Voir [[Documentation 02 — Gouvernance et Definition of Done]].

---

## 🪤 Piège 2 — Documentation hors Definition of Done

```
❌ "On mettra à jour la doc plus tard"
✅ La tâche n'est pas "Done" tant que la documentation associée n'est pas à jour
```

> [!tip] Mémo
> Ce qui n'est pas une condition de fin de tâche est systématiquement reporté puis oublié.

---

## 🪤 Piège 3 — Mélanger les familles de documentation dans un même document

```
❌ Un wiki d'équipe unique : architecture + procédures d'urgence + décisions passées, tout mélangé
✅ Documents séparés par famille (Cartographie, Architecture, Procédures, Référentiel, Historique)
```

> [!warning] Le mélange empêche des cycles de review différenciés
> Voir [[Documentation 03 — Les 5 familles de documentation]].

---

## 🪤 Piège 4 — Absence de cycles de review réguliers

```
❌ Le document est écrit une fois, jamais relu
✅ Review trimestrielle (runbooks critiques) ou semestrielle (overviews), + après chaque incident réel
```

> [!warning] Une documentation non relue devient silencieusement fausse
> Un runbook obsolète est pire qu'aucun runbook : il inspire une confiance non méritée en pleine gestion d'incident.

---

## 🪤 Piège 5 — Choisir l'outil avant la gouvernance

```
❌ Déployer un wiki ou un site Docs-as-Code, puis se demander qui écrit quoi
✅ Définir owners, familles et cycles de review, puis choisir l'outil qui les sert le mieux
```

> [!warning] L'outil ne résout pas le problème de responsabilité
> Voir [[Documentation 05 — Outils et maturité (Docs-as-Code vs Wiki)]].

---

## 🪤 Piège 6 — Postmortem qui cherche un coupable

```
❌ "Qui a fait l'erreur de déploiement ?"
✅ "Qu'est-ce qui, dans le processus, a permis cette erreur ?"
```

> [!warning] La chasse au coupable pousse à cacher les incidents
> Voir [[Documentation 04 — Documents essentiels (Overview, Runbook, Postmortem)]].

---

## 🪤 Piège 7 — Viser l'exhaustivité avant l'urgent

```
❌ Vouloir tout documenter dès le départ, sans jamais finir
✅ Urgence (services critiques) → Fondations → Maturité, dans cet ordre
```

> [!tip] Mémo
> Une documentation partielle mais à jour sur les services critiques vaut mieux qu'une documentation exhaustive mais figée.

---

---

## 🪤 Piège 8 — Wiki et Git documentaire maintenus en parallèle

```
❌ Une partie de l'équipe édite le wiki, une autre édite les fichiers Git — deux versions divergent
✅ Une seule source de vérité ; l'autre est mise en lecture seule puis fermée
```

> [!warning] Deux sources de vérité = aucune source fiable
> Voir [[Documentation 06 — Docs-as-Code, workflow et outillage]].

---

## 🪤 Piège 9 — Trop de friction pour une contribution mineure

```
❌ Exiger une Pull Request complète avec review pour corriger une simple typo
✅ Autoriser les commits directs pour les corrections mineures
```

> [!tip] Mémo
> Si contribuer demande plus d'effort que se taire, les gens arrêtent de contribuer. Voir [[Documentation 06 — Docs-as-Code, workflow et outillage]].

---

## 🪤 Piège 10 — Review effectuée par l'auteur seul

```
❌ L'auteur relit et valide son propre document
✅ Review croisée obligatoire, par quelqu'un d'autre que l'auteur
```

> [!warning] Biais de confirmation
> Voir [[Documentation 07 — Reviews et fraîcheur documentaire]].

---

## 🪤 Piège 11 — Supprimer une vieille documentation plutôt que l'archiver

```
❌ Suppression définitive d'un document obsolète
✅ Déplacement vers /archives/ avec bandeau explicite ("Référence historique uniquement")
```

> [!warning] La suppression détruit l'historique, l'archivage le préserve
> Voir [[Documentation 07 — Reviews et fraîcheur documentaire]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Document sans owner nommé | Désigner un owner explicite |
| Documentation hors Definition of Done | L'intégrer comme critère de fin de tâche |
| Familles mélangées dans un document fourre-tout | Séparer par fonction (Cartographie, Architecture, Procédures, Référentiel, Historique) |
| Pas de cycle de review régulier | Trimestriel/semestriel selon criticité + après incident |
| Outil choisi avant la gouvernance | Définir owners et cycles avant de choisir l'outil |
| Postmortem accusateur | Se concentrer sur les causes systémiques, pas les individus |
| Exhaustivité visée en premier | Prioriser les services critiques (trajectoire Urgence → Fondations → Maturité) |
| Wiki et Git maintenus en parallèle | Une seule source de vérité, l'autre fermée |
| Trop de friction pour une contribution mineure | Autoriser les commits directs pour les corrections mineures |
| Review effectuée par l'auteur seul | Review croisée obligatoire |
| Suppression d'une vieille documentation | Archivage avec bandeau explicite |
