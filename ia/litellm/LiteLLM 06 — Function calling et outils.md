#ia #litellm #agents #avancé

## Le contrat : un schéma décrit, un appel structuré en retour

Un LLM ne « appelle » jamais réellement une fonction — il produit du texte. Le function calling est le protocole qui transforme cette capacité en appels structurés : vous décrivez vos outils au modèle (nom, description, schéma JSON des paramètres), et quand il juge un outil pertinent, il ne l'exécute pas — il renvoie un objet structuré (nom + arguments JSON) que votre code doit exécuter lui-même. Voir [[Agents 03 — Les outils (Tools)]] pour ce principe de fonctionnement général.

> [!warning] Le schéma dit ce qu'on attend, il ne contraint rien
> Le modèle remplit le schéma « au mieux », sans garantie : il peut omettre un champ requis, inverser deux paramètres, ou passer un nombre sous forme de chaîne. D'où la nécessité de toujours valider ce qu'il renvoie avant d'exécuter quoi que ce soit.

## Pydantic comme source unique de vérité

Écrire le schéma JSON à la main dans un dictionnaire est une erreur : ce schéma vivrait à côté du code de l'outil, et les deux divergeraient à la première modification. Un modèle Pydantic décrit les arguments une seule fois, et sert deux fois : générer le schéma envoyé au LLM, et valider les arguments qu'il renvoie.

```python
from pydantic import BaseModel, Field

class MeteoArgs(BaseModel):
    """Arguments de l'outil météo."""
    ville: str = Field(min_length=1, description="Nom de la ville, ex : Paris")

SCHEMA = [{
    "type": "function",
    "function": {
        "name": "meteo",
        "description": "Donne la météo actuelle d'une ville.",
        "parameters": MeteoArgs.model_json_schema(),  # généré, jamais écrit à la main
    },
}]
```

> [!tip] Le jour où l'outil gagne un paramètre
> Ajoutez le champ à la classe Pydantic — le schéma envoyé au modèle et la validation des arguments suivent automatiquement, sans rien dupliquer entre les deux.

## L'outil ne lève jamais d'exception vers la boucle

```python
def meteo(ville: str) -> str:
    """Renvoie la météo actuelle d'une ville via une API externe."""
    try:
        geo = httpx.get(URL_GEOCODAGE, params={"name": ville, "count": 1}, timeout=10).json()
    except httpx.HTTPError:
        return "Service de géolocalisation indisponible."  # échec technique

    lieux = geo.get("results")
    if not lieux:
        return f"Ville introuvable : {ville}"  # échec « métier », pas technique
    # ... seconde requête pour la météo, même prudence ...
```

> [!warning] L'erreur est une donnée, pas un crash
> Un agent qui reçoit « Ville introuvable : Pariss » peut corriger la faute et réessayer au tour suivant. Une exception non gérée, elle, casse la boucle et l'agent s'arrête net. Que l'échec soit technique (API en panne) ou métier (ville qui n'existe pas), l'outil renvoie toujours un texte lisible — jamais une exception.

## Valider avant d'exécuter : le sas entre le modèle et le code

Entre l'appel du modèle et l'exécution réelle de l'outil, une étape filtre systématiquement tout ce qui peut mal se passer.

```python
def executer_outil(nom: str, arguments_json: str) -> str:
    """Valide les arguments du LLM avec Pydantic, puis exécute l'outil."""
    try:
        donnees = json.loads(arguments_json)
    except json.JSONDecodeError:
        return "Erreur : arguments JSON invalides."

    if nom != "meteo":
        return f"Erreur : outil inconnu « {nom} »."

    try:
        args = MeteoArgs(**donnees)  # validation Pydantic
    except ValidationError as erreur:
        detail = erreur.errors()[0]
        return f"Erreur de validation ({detail['loc']}) : {detail['msg']}."

    return meteo(args.ville)
```

Trois pannes réelles du function calling sont couvertes ici : le **JSON cassé** (un petit modèle produit parfois un objet mal formé), l'**argument halluciné** (le modèle appelle l'outil sans le champ requis, ou avec une valeur invalide — Pydantic le rejette net), et l'**outil inconnu** (le modèle invente un nom). Dans les trois cas, la fonction renvoie un texte qui repart au modèle, qui voit son erreur et se corrige au tour suivant.

> [!warning] L'ordre est non négociable : parser, valider, puis exécuter
> Appeler l'outil directement avec ce que renvoie le modèle revient à passer des arguments non vérifiés à du code qui touche au réseau ou au système. La validation Pydantic est la frontière entre le texte généré par le modèle et l'exécution réelle — ne jamais la sauter, même pour un outil qui semble anodin.

## La boucle via LiteLLM

LiteLLM normalise le schéma des outils et le format des `tool_calls` en retour, quel que soit le provider derrière — Ollama en local comme une API distante (voir [[LiteLLM 00 — Qu'est-ce que LiteLLM]]).

```python
from litellm import completion

reponse = completion(model=MODELE, messages=messages, tools=SCHEMA)
message = reponse.choices[0].message
```

Les arguments arrivent toujours en chaîne JSON dans `tool_calls`, d'où le passage systématique par `executer_outil`. Chaque résultat est réinjecté avec le rôle `tool`, rattaché à son appel via `tool_call_id` :

```python
for appel in message.tool_calls:
    resultat = executer_outil(appel.function.name, appel.function.arguments)
    messages.append(
        {"role": "tool", "tool_call_id": appel.id, "content": resultat}
    )
```

> [!info] `tool_call_id` importe dès que le modèle appelle plusieurs outils en un tour
> Si `message.tool_calls` contient plusieurs appels, chaque résultat doit être rattaché au bon `tool_call_id` — sans ce lien, le modèle ne peut pas savoir quel résultat correspond à quel appel.

> [!warning] Avec Ollama, utiliser `ollama_chat/` et non `ollama/`
> LiteLLM expose deux providers Ollama. `ollama/` est l'intégration historique ; `ollama_chat/` passe par l'endpoint `/api/chat`, seul à gérer correctement le va-et-vient des messages de rôle `tool`. Avec `ollama/`, un agent function-calling reboucle sur le même outil sans jamais produire de réponse finale — un symptôme qui ressemble à un bug de la boucle, mais qui vient du provider.

## Tester les trois faces du function calling

| Ce qui est testé | Nature | Dépend d'un LLM ? |
|---|---|---|
| L'outil interrogé en direct (cas normal + ville inexistante) | Test d'intégration contre l'API réelle | Non |
| La validation (JSON cassé, champ manquant) | Test déterministe de `executer_outil` | Non |
| L'agent complet sur une vraie question | Test d'intégration bout en bout | Oui |

Les deux premières catégories ne dépendent d'aucun modèle : elles sont rapides et déterministes. Seule la troisième appelle réellement le LLM — le réel prouve l'intégration, le déterministe prouve la robustesse. Voir [[Agents — Pièges classiques]] (Piège 8) pour ce même principe appliqué à la boucle agent elle-même.

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| L'agent reboucle sur le même outil | Provider LiteLLM `ollama/` | Utiliser `ollama_chat/<modèle>` |
| `ValidationError` non gérée, crash de l'agent | L'outil appelé sans passer par la validation | Parser et valider avec Pydantic avant d'exécuter |
| `json.JSONDecodeError` | Le modèle a produit des arguments mal formés | Capturer l'erreur, renvoyer un message au modèle |
| Le schéma et l'outil divergent au fil du temps | Schéma JSON écrit à la main | Générer le schéma depuis `model_json_schema()` |
| Aucun `tool_calls` en retour | Modèle sans support fiable du function calling | Choisir un modèle compatible (llama3.2 et plus récents) |

## Pour aller plus loin

Ce guide construit un outil isolé et sa validation ; brancher cette mécanique dans la boucle complète d'un agent est couvert par [[Agents 02 — Architecture d'un agent]] et [[Agents 03 — Les outils (Tools)]]. Pour ne plus réécrire cette boucle à la main et garantir la sortie de l'agent lui-même (pas seulement ses appels d'outils), voir [[PydanticAI — Index des fiches]].

Sources : [Function calling et outils — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/agents-function-calling/)
