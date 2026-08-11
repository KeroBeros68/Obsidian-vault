#ia #agents #multi-agents #avancé

## Multi-agents

Un système multi-agents est un ensemble d'agents IA qui **collaborent**, chacun avec un rôle spécialisé, pour accomplir un objectif commun.

## Pourquoi plusieurs agents ?

```
Agent unique sur une tâche complexe :
  → Context window saturée
  → Un seul LLM ne peut pas être expert en tout
  → Pas de parallélisme
  → Erreur à une étape = tout recommencer

Système multi-agents :
  → Chaque agent a un contexte ciblé et léger
  → Chaque agent est spécialisé dans son domaine
  → Certains agents travaillent en parallèle
  → Les erreurs sont isolées par agent
```

## Les 3 architectures de référence

Trois manières d'organiser une équipe d'agents reviennent partout dans les frameworks (LangGraph, CrewAI, AutoGen) — les connaître évite de réinventer une coordination bancale.

| Architecture | Coordination | Quand la choisir |
|---|---|---|
| **Superviseur** | Centralisée : un orchestrateur central décide de la suite, les agents ne se parlent jamais entre eux | Le défaut — la plus lisible et la plus facile à déboguer |
| **Swarm** (essaim) | Distribuée : chaque agent peut passer la main directement à un autre quand il juge que ce n'est plus son rôle | Parcours ouverts, peu prévisibles à l'avance |
| **Hiérarchique** | Superviseurs imbriqués : un superviseur de haut niveau orchestre des équipes, chacune étant elle-même un graphe avec son propre superviseur | Beaucoup d'agents, une équipe plate devenue trop large |

> [!info] Le superviseur est le bon point de départ
> C'est l'architecture la plus répandue : un seul point détient toute la logique de coordination, ce qui la rend lisible et testable. Le swarm gagne en souplesse ce qu'il perd en lisibilité — la logique de coordination se disperse dans tous les agents. Le hiérarchique n'est utile que face à un passage à l'échelle réel : composer des superviseurs plutôt que d'en avoir un seul devenu trop complexe.

Les architectures ci-dessous illustrent des variantes de ces trois patterns.

### Architecture 1 — Superviseur (Orchestrateur + Sous-agents)

```
[Agent Orchestrateur]
  "Prépare une analyse de marché complète"
        ↓
   ┌────┴─────┬──────────┐
   ↓          ↓          ↓
[Agent     [Agent     [Agent
Recherche] Analyse]   Rédaction]
   ↓          ↓          ↓
   └────┬─────┴──────────┘
        ↓
[Orchestrateur synthétise]
        ↓
  Résultat final
```

**Avantage** : coordination claire, l'orchestrateur garde le contrôle.

### Architecture 2 — Pipeline séquentiel

```
[Agent 1 Collecte] → Données brutes
    ↓
[Agent 2 Nettoyage] → Données propres
    ↓
[Agent 3 Analyse] → Insights
    ↓
[Agent 4 Rapport] → Document final
```

**Avantage** : simple, chaque agent reçoit exactement ce dont il a besoin.

### Architecture 3 — Pair-à-pair (collaboration)

```
[Agent A] ←──────────────→ [Agent B]
"Expert technique"         "Expert business"
         ↓ débattent et itèrent
    [Consensus / Décision]
```

Utilisé par AutoGen pour la génération de code (un agent code, un autre relit et critique).

### Architecture 4 — Réseau d'agents spécialisés

```
[Router Agent]
    ↓ analyse la demande et route
    ├──→ [Agent Juridique]    si question légale
    ├──→ [Agent Financier]    si question finance
    ├──→ [Agent Technique]    si question tech
    └──→ [Agent Commercial]   si question vente
```

> [!info] Les architectures 1 et 4 sont deux variantes du pattern Superviseur
> Dans les deux cas, un point central décide et les agents spécialisés ne se parlent jamais directement entre eux — la seule différence est que l'architecture 1 revient systématiquement à l'orchestrateur après chaque agent, quand l'architecture 4 route une fois puis termine. Voir plus bas « Implémenter un superviseur avec LangGraph » pour une implémentation complète de ce pattern, avec boucle de correction.

### Architecture 5 — Swarm (essaim)

```
[Agent Rédaction] ──passe la main──→ [Agent Traduction]
        ↑                                    │
        │                          passe la main si besoin
        └──────── [Agent Relecture] ◄────────┘

Chaque agent décide lui-même à qui transmettre —
aucun orchestrateur central ne détient la logique de coordination.
```

**Avantage** : souple face à un parcours imprévisible, aucun point de passage obligé. **Inconvénient** : la logique de coordination se disperse dans chaque agent, ce qui rend l'ensemble plus difficile à suivre et à déboguer qu'un superviseur centralisé.

### Architecture 6 — Hiérarchique (superviseurs imbriqués)

```
                [Superviseur principal]
                   ↓              ↓
        [Équipe Rédaction]   [Équipe Recherche]
        (son propre           (son propre
         superviseur)          superviseur)
           ↓      ↓              ↓      ↓
        [Agent] [Agent]       [Agent] [Agent]
```

Chaque équipe est elle-même un graphe superviseur complet, orchestré à son tour par un superviseur de niveau supérieur. Utile quand une équipe plate (architecture 1) devient trop large pour rester lisible — on la découpe en sous-équipes, chacune avec sa propre coordination.

## Exemple concret : pipeline de création de contenu

```
[Brief] → Agent Recherche → collecte infos sur le sujet
              ↓
          Agent SEO → identifie les mots-clés cibles
              ↓
          Agent Rédaction → rédige l'article
              ↓
          Agent Éditorial → vérifie le style et la cohérence
              ↓
          Agent Formatage → met en forme pour WordPress
              ↓
          [Article prêt à publier]
```

## Communication entre agents

Les agents se passent des informations via :

| Mécanisme | Description |
|---|---|
| **Message passing** | Un agent envoie un message structuré à un autre |
| **État partagé** | Tous les agents lisent/écrivent dans un état commun |
| **File de tâches** | Les agents prennent des tâches dans une queue |
| **Mémoire partagée** | Vector DB commune accessible à tous les agents |

## Implémenter un superviseur avec LangGraph

Le pattern Superviseur se construit avec exactement le modèle de graphe vu dans [[LC 12 — LangGraph — agents avec état]] : un état partagé, des nœuds, des arêtes. Exemple : une équipe de rédaction — un rédacteur, un relecteur, un fact-checker — coordonnée par un superviseur.

### L'état partagé, seul canal entre les agents

```python
from typing_extensions import TypedDict

class EtatRedaction(TypedDict):
    sujet: str            # le thème du brouillon à produire
    brouillon: str        # écrit par le rédacteur
    revue: str            # notes de style du relecteur
    verdict: str          # "ok" ou "problemes", rendu par le fact-checker
    commentaire: str      # justification du fact-checker
    revisions: int        # nombre de passes du rédacteur
    prochaine_etape: str  # décidée par le superviseur
```

> [!info] Le découplage par l'état est ce qui rend l'équipe extensible
> Le relecteur ne connaît pas le rédacteur : il lit `brouillon`, écrit `revue`. Ajouter un agent, c'est ajouter un nœud qui lit et écrit des clés de l'état, sans toucher au code des autres agents.

### Un agent avec sortie structurée pour un verdict exploitable

Le fact-checker mérite une attention particulière : son verdict doit être exploitable par le superviseur, pas un paragraphe à interpréter. On lui impose une sortie structurée (voir [[PydanticAI 00 — Qu'est-ce que PydanticAI]] pour ce même principe, ici via `with_structured_output` — voir [[LC 03 — LLM et Chat Models]]).

```python
from typing import Literal
from pydantic import BaseModel, Field

class Verification(BaseModel):
    verdict: Literal["ok", "problemes"] = Field(
        description="ok si les affirmations du brouillon tiennent"
    )
    commentaire: str = Field(description="Justification du verdict, une phrase")

def fact_checker(etat: EtatRedaction) -> dict:
    """Agent fact-checker : vérifie les faits et rend un verdict structuré."""
    verif = LLM.with_structured_output(Verification).invoke([
        ("system", "Tu es un fact-checker. Tu vérifies si les affirmations du texte sont exactes."),
        ("human", f"Vérifie les faits de ce brouillon :\n{etat['brouillon']}"),
    ])
    return {"verdict": verif.verdict, "commentaire": verif.commentaire}
```

### Le superviseur : une fonction de décision pure, sans appel modèle

Le superviseur est le cœur du pattern — un nœud qui ne produit aucun contenu, il décide seulement quel agent intervient ensuite, en lisant l'état.

```python
MAX_REVISIONS = 2

def superviseur(etat: EtatRedaction) -> dict:
    """Décide quel agent intervient ensuite : le cœur du pattern."""
    if not etat.get("brouillon"):
        etape = "redacteur"
    elif not etat.get("revue"):
        etape = "relecteur"
    elif not etat.get("verdict"):
        etape = "fact_checker"
    elif etat["verdict"] == "problemes" and etat["revisions"] <= MAX_REVISIONS:
        etape = "redacteur"  # le fact-checker a trouvé un problème : on révise
    else:
        etape = "FIN"
    return {"prochaine_etape": etape}
```

Sa logique se lit comme une liste de priorités : pas de brouillon → le rédacteur ; un brouillon mais pas de revue → le relecteur ; et ainsi de suite. Toute la coordination de l'équipe tient dans cette seule fonction — pour la comprendre, on lit le superviseur, pas les trois agents.

> [!tip] Superviseur déterministe ou piloté par un modèle
> Ce superviseur applique une règle déterministe : suffisant, fiable, trivial à tester. On peut aussi confier la décision à un modèle — utile quand le routage est réellement ouvert (« choisis l'agent le plus pertinent pour cette demande »). Le coût : une décision moins prévisible, à sécuriser par une sortie structurée. Pour un enchaînement balisé comme celui d'une équipe de rédaction, la règle déterministe est le bon choix.

### Boucler pour corriger, avec un plafond obligatoire

Quand le fact-checker rend le verdict `problemes`, le superviseur renvoie au rédacteur, qui révise en tenant compte du commentaire — une boucle de correction, l'équipe itère jusqu'à ce que les faits tiennent.

> [!warning] Tout système qui boucle a besoin d'une condition d'arrêt
> Sans garde-fou, un fact-checker exigeant pourrait renvoyer sans fin. `MAX_REVISIONS` borne la boucle : passé ce nombre de passes, le superviseur conclut, même si le verdict reste imparfait — le même principe de guardrail que dans [[Agents 02 — Architecture d'un agent]] (guardrails) et [[Agents — Pièges classiques]] (Piège 1), appliqué ici à une boucle inter-agents plutôt qu'à une boucle d'outils.

### Le graphe : topologie en étoile

Chaque agent a une arête fixe vers le superviseur (il revient toujours au centre) ; le superviseur, lui, a des arêtes conditionnelles vers l'agent que le routeur désigne — cette topologie en étoile matérialise le pattern Superviseur.

```python
from langgraph.graph import END, START, StateGraph

def router(etat: EtatRedaction):
    """Traduit la décision du superviseur en destination du graphe."""
    etape = etat["prochaine_etape"]
    return END if etape == "FIN" else etape

constructeur = StateGraph(EtatRedaction)
constructeur.add_node("superviseur", superviseur)
constructeur.add_node("redacteur", redacteur)
constructeur.add_node("relecteur", relecteur)
constructeur.add_node("fact_checker", fact_checker)

constructeur.add_edge(START, "superviseur")
constructeur.add_conditional_edges(
    "superviseur", router, ["redacteur", "relecteur", "fact_checker", END]
)
constructeur.add_edge("redacteur", "superviseur")
constructeur.add_edge("relecteur", "superviseur")
constructeur.add_edge("fact_checker", "superviseur")

equipe = constructeur.compile()
```

Le nombre de passes varie d'une exécution à l'autre — le fact-checker peut accepter le brouillon du premier coup ou exiger plusieurs révisions — mais le graphe s'arrête toujours, sur un verdict `ok` ou au plafond `MAX_REVISIONS`.

> [!tip] Tester le superviseur sans modèle
> Le routage du superviseur se teste sans modèle, de façon déterministe : lui passer des états et vérifier sa décision, y compris l'arrêt au plafond de révisions. Seul un test d'intégration bout en bout (l'équipe complète produit bien un brouillon, une revue et un verdict) justifie un appel réel au LLM — même principe que dans [[Agents — Pièges classiques]] (Piège 8).

## Frameworks multi-agents

| Framework | Architecture | Points forts |
|---|---|---|
| **AutoGen** (Microsoft) | Pair-à-pair, conversations | Excellent pour code et debug |
| **CrewAI** | Hiérarchique avec rôles | Simple, intuitif, bon pour débuter |
| **LangGraph** | Graphe d'états | Très flexible, contrôle total |
| **AgentScope** (Alibaba) | Pipeline distribué | Tolérance aux pannes |

## Forces et limites

| ✅ Points forts | ❌ Points faibles |
|---|---|
| Parallélisme → plus rapide | Complexité de coordination |
| Spécialisation → meilleure qualité | Coût élevé en tokens |
| Modularité → facile à étendre | Débogage très difficile |
| Contextes plus légers par agent | Risques de boucles et conflits |

> [!warning] Ne pas sur-architecturer
> Commence toujours par un seul agent. Passe au multi-agents uniquement si tu rencontres des limites claires : context window insuffisante, besoin de parallélisme, ou domaines trop hétérogènes pour un seul agent.

> [!tip] CrewAI pour débuter
> CrewAI est le framework le plus accessible pour découvrir les multi-agents. Tu définis des agents avec des rôles en langage naturel, et il gère la coordination automatiquement.

## Dépannage (pattern Superviseur avec LangGraph)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `GraphRecursionError` | La boucle de correction ne s'arrête jamais | Vérifier la condition d'arrêt et le plafond (`MAX_REVISIONS`) |
| Un agent ne voit pas le travail d'un autre | Clé d'état mal nommée, ou jamais écrite par le nœud censé la remplir | Vérifier les clés lues et renvoyées par chaque nœud |
| Le superviseur saute une étape | Ordre des conditions incorrect dans la fonction de décision | Ordonner les tests du plus précoce au plus tardif dans le parcours |
| La révision ignore le retour d'un agent précédent | Le commentaire n'est pas transmis à l'agent qui doit corriger | Lire la clé d'état correspondante (ex. `etat["commentaire"]`) dans le nœud qui révise |
| Le verdict d'un agent de contrôle est du texte libre | Sortie non structurée | Utiliser `with_structured_output` avec un `Literal`, voir plus haut |
