#ia #dspy #configuration #lm #bases

## Configuration et LM

## Installation

```bash
pip install dspy

# Avec extras pour les embeddings et optimiseurs avancés
pip install "dspy[all]"
```

## Configurer le LM

DSPy utilise LiteLLM sous le capot — il supporte nativement tous les providers.

```python
import dspy

# ── Claude (Anthropic) ────────────────────────────────────
lm = dspy.LM("anthropic/claude-sonnet-4-20250514", api_key="ta-clé")
dspy.configure(lm=lm)

# ── GPT-4o (OpenAI) ───────────────────────────────────────
lm = dspy.LM("openai/gpt-4o", api_key="ta-clé")
dspy.configure(lm=lm)

# ── Gemini (Google) ───────────────────────────────────────
lm = dspy.LM("google/gemini-2.0-flash", api_key="ta-clé")
dspy.configure(lm=lm)

# ── Modèle local via Ollama ───────────────────────────────
lm = dspy.LM("ollama/mistral", api_base="http://localhost:11434")
dspy.configure(lm=lm)

# ── Modèle local via vLLM ─────────────────────────────────
lm = dspy.LM("openai/Qwen/Qwen3-8B", api_base="http://localhost:8000/v1", api_key="no-key")
dspy.configure(lm=lm)
```

## Paramètres du LM

```python
lm = dspy.LM(
    "anthropic/claude-sonnet-4-20250514",
    api_key="ta-clé",
    temperature=0.7,        # créativité (0=déterministe, 1=créatif)
    max_tokens=2000,        # limite de génération
    cache=True,             # mettre en cache les réponses (économise les coûts)
    num_retries=3           # retries en cas d'erreur API
)
dspy.configure(lm=lm)
```

## Utiliser plusieurs LM dans un même programme

```python
import dspy

# LM principal (puissant, pour les tâches complexes)
lm_puissant = dspy.LM("anthropic/claude-sonnet-4-20250514")

# LM léger (rapide, pour les tâches simples)
lm_léger = dspy.LM("anthropic/claude-haiku-4-5")

# Configurer le LM par défaut
dspy.configure(lm=lm_puissant)

# Dans un module, overrider localement
class MonProgramme(dspy.Module):
    def __init__(self):
        # Ce module utilise le LM léger pour classifier
        self.classifieur = dspy.Predict("texte -> catégorie")
        # Ce module utilise le LM puissant pour analyser
        self.analyseur   = dspy.ChainOfThought("texte, catégorie -> analyse_détaillée")

    def forward(self, texte):
        with dspy.context(lm=lm_léger):
            catégorie = self.classifieur(texte=texte)
        # Utilise le LM par défaut (puissant)
        analyse = self.analyseur(texte=texte, catégorie=catégorie.catégorie)
        return analyse
```

## Inspecter les appels LM

```python
import dspy

lm = dspy.LM("anthropic/claude-sonnet-4-20250514")
dspy.configure(lm=lm)

# Faire un appel
predict = dspy.Predict("question -> réponse")
résultat = predict(question="Qu'est-ce que le RAG ?")

# Inspecter l'historique des appels (prompts envoyés + réponses)
dspy.inspect_history(n=1)
# → Affiche le prompt exact envoyé au LLM + la réponse brute
# Très utile pour comprendre ce que DSPy fait sous le capot
```

## Caching — économiser les coûts en développement

```python
import dspy

# Cache activé par défaut — les mêmes requêtes ne sont appelées qu'une fois
lm = dspy.LM("anthropic/claude-sonnet-4-20250514", cache=True)
dspy.configure(lm=lm)

# Désactiver le cache pour les tests de variabilité
lm_no_cache = dspy.LM("anthropic/claude-sonnet-4-20250514", cache=False)

# Le cache est stocké localement dans ~/.dspy_cache/
# Vider le cache si besoin
import diskcache as dc
cache = dc.Cache("~/.dspy_cache")
cache.clear()
```

## Configurer les embeddings (pour le RAG)

```python
import dspy

# Embeddings pour la recherche vectorielle
embedder = dspy.Embedder(
    "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
)

# Ou avec OpenAI
embedder = dspy.Embedder("openai/text-embedding-3-small", api_key="ta-clé")
```

> [!tip] Toujours activer le cache en développement
> `cache=True` est le défaut et c'est une bonne chose. Pendant le développement, tu vas répéter les mêmes appels des centaines de fois. Le cache économise du temps et de l'argent.

> [!info] dspy.inspect_history() — ton meilleur ami pour déboguer
> Quand DSPy ne fait pas ce que tu attends, inspecte le prompt exact qu'il génère. DSPy construit des prompts sophistiqués sous le capot — les voir explique souvent le comportement inattendu.
