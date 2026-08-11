#ia #pydanticai #pièges #erreurs #debugging

## 🪤 Piège 1 — Confondre les trois modes de sortie structurée

```python
# ❌ Tool Output par défaut sur un petit modèle 3B faible en function calling
agent = Agent(modele_3b, output_type=Synthese)  # échoue après tous les retries

# ✅ Diagnostiquer d'abord le mode actif, puis adapter au modèle
agent = Agent(modele_3b, output_type=Synthese, retries=5)  # augmenter le budget
# ou, en dernier recours :
agent = Agent(modele_3b, output_type=Synthese, output_mode="prompted")
```

> [!warning] Chercher au mauvais endroit fait perdre des heures
> En Tool Output, un échec vient du schéma de l'outil de sortie ou de la capacité du modèle à l'appeler correctement. En Native Output, il vient du support (souvent absent) du modèle. En Prompted Output, il vient du prompt ou du parsing. Identifier le mode actif avant de déboguer. Voir [[PydanticAI 01 — Sortie structurée (Tool, Native, Prompted)]].

---

## 🪤 Piège 2 — Variables globales à la place des dépendances

```python
# ❌ Client HTTP global : impossible à mocker en test, à réutiliser dans un autre contexte
client = httpx.Client()

@agent.tool
def lister_releases(ctx: RunContext[None], depot: str) -> list[dict]:
    return recuperer_releases(client, depot)  # dépend d'un état hors de l'agent

# ✅ Injection via deps_type et RunContext
@agent.tool
def lister_releases(ctx: RunContext[Deps], depot: str) -> list[dict]:
    return recuperer_releases(ctx.deps.client, depot)
```

> [!tip] Voir [[PydanticAI 02 — Dépendances, outils et ModelRetry]]
> Un agent dont les outils dépendent d'un état global ne peut pas être testé isolément (impossible d'injecter un client factice) ni réutilisé avec une configuration différente sans modifier le code.

---

## 🪤 Piège 3 — Lever ModelRetry hors d'un outil ou d'un validateur

```python
# ❌ ModelRetry levée dans le code appelant, pas dans l'outil lui-même
def resumer(depot: str):
    if "/" not in depot:
        raise ModelRetry("format invalide")  # jamais interceptée par PydanticAI
    ...

# ✅ ModelRetry vaut uniquement à l'intérieur d'un outil décoré @agent.tool
@agent.tool
def lister_releases(ctx: RunContext[Deps], depot: str) -> list[dict]:
    if "/" not in depot:
        raise ModelRetry("Le dépôt doit être au format owner/repo.")
    return recuperer_releases(ctx.deps.client, depot)
```

> [!warning] Hors de ce contexte, ModelRetry se comporte comme une exception normale
> Elle n'est interceptée et transformée en boucle de correction que lorsqu'elle est levée depuis l'intérieur d'un outil décoré ou d'une fonction de validation de sortie — ailleurs, elle remonte et interrompt l'exécution comme n'importe quelle autre exception.

---

## 🪤 Piège 4 — Tester uniquement l'agent complet contre un vrai modèle

```python
# ❌ Un seul test, lent et non déterministe, pour tout couvrir
def test_agent():
    resultat = agent.run_sync("Résume astral-sh/uv", deps=Deps(client=httpx.Client()))
    assert resultat.output.derniere_version  # dépend du réseau ET du modèle

# ✅ Séparer logique métier, retry, et sortie typée (voir PydanticAI 02)
def test_lister_releases_format_invalide():
    with pytest.raises(ModelRetry):
        lister_releases(ctx_factice, depot="pas-de-slash")  # sans réseau ni modèle
```

> [!tip] Deux des trois contrôles ne dépendent d'aucun modèle
> La logique métier (appel API réel) et le déclenchement de `ModelRetry` sont testables de façon déterministe, sans jamais appeler le LLM. Réserver le test contre le vrai modèle à la vérification de bout en bout — le même principe que pour un agent écrit à la main (voir [[Agents — Pièges classiques]], Piège 8).

---

## Récapitulatif rapide

| Piège | Solution |
|---|---|
| Confondre les 3 modes de sortie | Identifier le mode actif avant de déboguer |
| Variables globales dans les outils | Injection via `deps_type` + `RunContext` |
| `ModelRetry` levée hors d'un outil | La lever uniquement dans un outil ou un validateur |
| Tests 100% dépendants du vrai modèle | Isoler logique métier et retry, réserver le réel à l'intégration |
