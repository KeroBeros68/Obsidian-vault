#ia #pydanticai #agents #pratique

## Injecter des dépendances plutôt que des globales

Un agent a souvent besoin de ressources externes — un client HTTP, une connexion, une configuration. Les coder en variables globales rend l'agent impossible à tester isolément et à réutiliser dans un autre contexte. PydanticAI répond par l'**injection de dépendances** : un objet, déclaré par `deps_type`, passé à chaque exécution et transmis automatiquement aux outils.

```python
from dataclasses import dataclass
import httpx

@dataclass
class Deps:
    """Dépendances de l'agent : ici, le client HTTP partagé."""
    client: httpx.Client
```

L'agent se déclare avec ses quatre paramètres (voir [[PydanticAI 00 — Qu'est-ce que PydanticAI]]), et le budget de retries couvre à la fois les outils et la sortie :

```python
agent = Agent(
    MODELE,
    deps_type=Deps,
    output_type=SyntheseRelease,
    retries=3,
    system_prompt=(
        "Tu résumes les notes de version d'un dépôt GitHub. Appelle l'outil "
        "lister_releases, puis produis une synthèse structurée et factuelle."
    ),
)
```

## Outils avec RunContext, et ModelRetry pour les erreurs

Un outil PydanticAI qui a besoin des dépendances se décore avec `@agent.tool` et reçoit un `RunContext` typé, par lequel il accède à `ctx.deps` — sans aucune variable globale.

```python
from pydantic_ai import ModelRetry, RunContext

@agent.tool
def lister_releases(ctx: RunContext[Deps], depot: str) -> list[dict]:
    """Liste les dernières releases d'un dépôt GitHub.

    depot : identifiant au format owner/repo, par exemple astral-sh/uv.
    """
    if "/" not in depot:
        raise ModelRetry("Le dépôt doit être au format owner/repo.")
    return recuperer_releases(ctx.deps.client, depot)
```

> [!info] ModelRetry transforme une erreur en boucle de correction, pas un crash
> Si l'outil lève `ModelRetry`, PydanticAI ne plante pas : il renvoie le message d'erreur au modèle, qui corrige son appel et réessaie, dans la limite du budget `retries`. Un argument mal formé devient ainsi une boucle de correction automatique — le même principe que renvoyer un message d'erreur exploitable plutôt qu'une exception, déjà vu pour les outils écrits à la main dans [[Agents 03 — Les outils (Tools)]] et [[LiteLLM 06 — Function calling et outils]].

> [!warning] ModelRetry ne vaut que dans un outil ou une fonction de sortie
> Levée ailleurs dans le code, l'exception n'est pas interceptée par PydanticAI et se comporte comme une erreur normale — elle doit être levée depuis l'intérieur d'un outil décoré ou d'un validateur de sortie pour déclencher la boucle de correction.

## Faire tourner l'agent

```python
def resumer(depot: str) -> SyntheseRelease:
    with httpx.Client(timeout=15) as client:
        resultat = agent.run_sync(
            f"Résume les dernières releases du dépôt {depot}.",
            deps=Deps(client=client),
        )
        return resultat.output
```

## Tester un agent PydanticAI : trois contrôles distincts

| Ce qui est testé | Nature | Dépend d'un modèle ? |
|---|---|---|
| La logique métier (l'outil interroge l'API réelle) | Test d'intégration, déterministe sur une ressource stable | Non |
| Le retry (`ModelRetry` levée sur un argument mal formé) | Test unitaire de l'outil seul | Non |
| La sortie typée (l'agent complet renvoie l'instance attendue) | Test d'intégration bout en bout | Oui |

Deux des trois contrôles ne dépendent d'aucun modèle — c'est la même discipline que pour un agent écrit à la main (voir [[Agents — Pièges classiques]], Piège 8) : isoler ce qui est déterministe, réserver l'appel réel au modèle au strict nécessaire.

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| `UnexpectedModelBehavior: Exceeded maximum output retries` | Le modèle ne produit pas une sortie valide | Augmenter `retries` ; si besoin, passer en Prompted Output (voir [[PydanticAI 01 — Sortie structurée (Tool, Native, Prompted)]]) |
| `Agent(output_retries=...)` signalé comme déprécié | Ancienne API | Utiliser `retries={'output': N}` ou `retries=N` |
| L'agent ne voit pas les dépendances | `deps` non passé à `run_sync`, ou `deps_type` absent | Déclarer `deps_type` et passer `deps=` à chaque exécution |
| Le modèle boucle sur le même outil | Modèle faible en function calling | Choisir un modèle solide (`qwen2.5`) ; relire sa documentation |
| `ModelRetry` non prise en compte | Levée hors d'un outil ou d'un validateur | `ModelRetry` ne vaut que dans un outil ou une fonction de sortie |

## Pour aller plus loin

Ceci conclut l'introduction à PydanticAI — voir [[PydanticAI — Index des fiches]] pour une vue d'ensemble, et [[Agents 07 — LangChain et LangGraph]] pour la comparaison avec les autres frameworks d'agents.

Sources : [Agents typés avec PydanticAI — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/agents-pydanticai/)
