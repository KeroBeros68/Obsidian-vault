#devops #documentation #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Bus factor** (truck factor) | Nombre de personnes qui peuvent disparaître d'une équipe avant que le fonctionnement ne soit bloqué faute de connaissance. |
| **Owner** | Personne responsable de la mise à jour et de l'exactitude d'un document. |
| **Contributeur** | Personne pouvant proposer des modifications à un document sans en être responsable. |
| **Reviewer** | Personne qui valide un changement avant son intégration au document. |
| **Definition of Done** | Ensemble de critères qu'une tâche doit remplir pour être considérée terminée (tests, review, documentation, CHANGELOG). |
| **Service Overview** | Document de synthèse d'un service : owner, criticité, SLA, architecture simplifiée, dépendances, contacts. |
| **Runbook** | Procédure d'exécution pas-à-pas pour une situation connue (déploiement, incident type), avec commandes exactes. |
| **Postmortem** | Analyse factuelle d'un incident après résolution : timeline, impact, causes racines, actions correctives — sans recherche de coupable. |
| **ADR** (Architecture Decision Record) | Document immuable actant une décision d'architecture et son contexte au moment où elle a été prise. |
| **5 Whys** | Méthode consistant à demander "pourquoi" successivement pour remonter d'un symptôme à sa cause racine. |
| **Checklist** | Liste de vérification courte (10-15 points) pour un moment à risque récurrent (déploiement, onboarding). |
| **SLA** (Service Level Agreement) | Engagement mesurable de disponibilité ou de performance d'un service. |
| **Docs-as-Code** | Approche où la documentation est écrite en Markdown et versionnée dans Git, avec review par pull request. |
| **Wiki** | Outil d'édition de documentation en ligne, WYSIWYG, accessible sans compétence technique (ex. Confluence, Notion). |
| **MkDocs** | Générateur de site de documentation technique à partir de fichiers Markdown, simple à mettre en place. |
| **Docusaurus** | Générateur de site de documentation orienté produit, avec gestion de versions par release. |
| **Antora** | Générateur de documentation agrégeant plusieurs dépôts/projets en un seul site. |
| **Reviewer naïf** | Relecteur qui ne connaît pas le sujet en détail — s'il comprend et peut suivre le document, c'est un bon signe de clarté. |
| **Vale** | Linter de prose configurable, applique un style guide (Microsoft, Google, ou personnalisé) au texte de documentation. |
| **markdownlint** | Outil validant la syntaxe et la structure d'un fichier Markdown (titres séquentiels, formatage). |
| **lychee** | Outil de détection des liens cassés dans une documentation, écrit en Rust. |
| **MDX** | Format combinant Markdown et composants interactifs (JSX), utilisé quand la documentation a besoin d'éléments dynamiques. |
| **SSG** (Static Site Generator) | Outil générant un site de documentation statique à partir de fichiers texte (ex. MkDocs, Docusaurus, Hugo). |
| **Freshness date** | Date et responsable de la dernière review d'un document, affichés pour en évaluer la fiabilité (terme employé par Google). |
| **Frontmatter** | Bloc de métadonnées en tête d'un fichier Markdown (ex. `last_reviewed`, `owner`, `criticality`), exploitable par des scripts d'automatisation. |
| **Biais de confirmation (review)** | Tendance d'un auteur à valider son propre document sans en voir les lacunes, faute de regard extérieur — d'où l'exigence d'une review croisée. |
