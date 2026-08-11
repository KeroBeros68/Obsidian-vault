#ia #pydanticai #agents #intermédiaire

## Trois façons d'obtenir une structure du modèle

[[PydanticAI 00 — Qu'est-ce que PydanticAI]] montre que la sortie d'un agent est un modèle Pydantic garanti. Pour l'obtenir, PydanticAI dispose de trois mécanismes distincts — les confondre, ou choisir le mauvais pour un modèle donné, coûte des heures de débogage.

| Mode | Mécanisme | Support | Fiabilité | Pour un modèle local |
|---|---|---|---|---|
| **Tool Output** (défaut) | Le schéma de sortie est décrit comme un outil que le modèle doit appeler pour livrer son résultat | Tous les modèles | Très haute | Recommandé |
| **Native Output** | S'appuie sur la fonction de *structured output* native du modèle, qui le contraint à respecter le schéma | Modèles sélectionnés seulement | Très haute | Rarement supporté |
| **Prompted Output** | Le schéma JSON est injecté dans les instructions ; la réponse texte est parsée | Tous les modèles | Modérée — rien ne contraint réellement le modèle | Solution de repli |

Le **Tool Output** est le mode par défaut de PydanticAI : « supporté par pratiquement tous les modèles », c'est celui que la documentation recommande pour un modèle local. Il suffit de passer la classe Pydantic à `output_type`, sans rien envelopper :

```python
agent = Agent(MODELE, output_type=SyntheseRelease, ...)
```

Le **Native Output** est le plus fiable en théorie — il s'appuie sur un vrai *constrained decoding* côté modèle (voir [[CD — Index des fiches]] pour ce mécanisme en général) — mais peu de modèles locaux l'exposent correctement : à éviter par défaut hors des grands modèles cloud qui le supportent explicitement.

Le **Prompted Output** fonctionne avec n'importe quel modèle, y compris ceux sans aucun support de function calling, mais c'est « souvent l'approche la moins fiable » : rien n'empêche le modèle de dévier du format demandé, et PydanticAI doit alors parser un texte qui n'est fiable qu'en apparence.

> [!tip] Le mode de sortie se choisit selon le modèle
> Un modèle solide en function calling (`qwen2.5`, par exemple) produit une sortie typée propre en Tool Output. Un modèle plus petit et plus faible (3B) peut échouer même là — la parade documentée est d'abord d'augmenter le budget de `retries`, et en dernier recours de passer en Prompted Output. Vérifier la documentation du modèle avant de choisir évite des heures de tâtonnement.

> [!warning] Confondre les trois modes est la première cause de blocage
> Chercher à déboguer une sortie mal formée sans savoir quel mode est actif revient à chercher au mauvais endroit : en Tool Output l'erreur vient du schéma de l'outil de sortie, en Native Output du support du modèle, en Prompted Output du prompt et du parsing. Identifier le mode actif est le premier réflexe de dépannage.

## Pour aller plus loin

Une fois la sortie typée acquise, la suite logique est d'alimenter l'agent en ressources externes sans variables globales — voir [[PydanticAI 02 — Dépendances, outils et ModelRetry]].

Sources : [Agents typés avec PydanticAI — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/agents-pydanticai/)
