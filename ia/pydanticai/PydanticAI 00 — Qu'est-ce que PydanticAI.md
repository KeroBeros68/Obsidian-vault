#ia #pydanticai #agents #bases

## Ce que PydanticAI change

Les deux guides précédents ([[Agents 02 — Architecture d'un agent]], [[LiteLLM 06 — Function calling et outils]]) écrivent la boucle d'un agent à la main — instructif, mais on ne la réécrit pas à chaque projet. PydanticAI fournit cette boucle toute prête, et lui ajoute ce qui sépare un script d'un composant de production : une **sortie typée garantie**, l'**injection de dépendances**, et des **retries automatiques** quand le modèle se trompe.

## Du texte libre à la sortie typée

Un agent écrit à la main renvoie une chaîne de caractères. Pour un affichage, c'est suffisant. Pour un composant qui s'insère dans une chaîne de traitement — alimenter une base, déclencher une action, être testé — c'est ingérable : il faut parser ce texte, espérer qu'il a la bonne forme, gérer le cas où il ne l'a pas.

PydanticAI renverse le problème : on déclare la sortie attendue comme un modèle Pydantic, et l'agent garantit de renvoyer une instance validée de ce modèle, **ou d'échouer franchement**. Plus de parsing, plus d'espoir — le type est une certitude.

> [!info] Le même principe que le function calling, appliqué à la réponse finale
> C'est exactement le mécanisme schéma + validation vu dans [[LiteLLM 06 — Function calling et outils]] pour un appel d'outil — PydanticAI l'applique cette fois à la sortie finale de l'agent lui-même, pas seulement à ses appels d'outils intermédiaires.

## Anatomie d'un agent PydanticAI

Un agent se déclare en un objet `Agent`, paramétré par quatre éléments : le **modèle**, le type des **dépendances** (`deps_type`), le type de **sortie** (`output_type`), et les **outils** (déclarés par décoration, pas au constructeur).

```python
from pydantic_ai import Agent
from pydantic_ai.models.ollama import OllamaModel
from pydantic_ai.providers.ollama import OllamaProvider

MODELE = OllamaModel(
    "qwen2.5",
    provider=OllamaProvider(base_url="http://localhost:11434/v1"),
)
```

PydanticAI fournit une classe dédiée pour un modèle Ollama auto-hébergé (voir [[Ollama — Index des fiches]]) : `OllamaModel`, branchée sur un `OllamaProvider`.

## La sortie typée en pratique

```python
from pydantic import BaseModel, Field

class SyntheseRelease(BaseModel):
    """Synthèse structurée des dernières releases d'un dépôt."""
    derniere_version: str = Field(description="Tag de la release la plus récente")
    nombre_de_releases: int = Field(description="Nombre de releases examinées")
    resume: str = Field(description="Résumé factuel des changements, 2 à 3 phrases")
```

Ce modèle devient le `output_type` de l'agent. Dès lors, `agent.run_sync(...)` renvoie un résultat dont le `.output` est une instance de `SyntheseRelease`, ou lève `UnexpectedModelBehavior` si le modèle n'y est pas parvenu, même après ses retries.

> [!warning] Il n'y a pas d'entre-deux
> Un appel qui réussit a produit une sortie valide et typée. Un appel qui échoue lève une exception explicite. Il n'existe aucun état intermédiaire où l'agent renverrait « quelque chose qui ressemble » à la sortie attendue — c'est cette garantie qui rend un agent PydanticAI branchable dans du code qui ne sait pas gérer l'incertitude d'un texte libre.

## Pour aller plus loin

Comment PydanticAI obtient-il cette structure du modèle ? Voir [[PydanticAI 01 — Sortie structurée (Tool, Native, Prompted)]] pour les trois mécanismes possibles et lequel choisir selon le modèle utilisé.

Sources : [Agents typés avec PydanticAI — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/agents-pydanticai/)
