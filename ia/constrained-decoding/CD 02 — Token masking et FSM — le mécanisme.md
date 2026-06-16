#ia #constrained-decoding #fsm #token-masking #mécanisme #intermédiaire

## Token masking et FSM — le mécanisme

Comprendre comment le constrained decoding fonctionne au niveau des tokens et des automates finis.

## La génération autoregressive — rappel

```
À chaque étape, le LLM :
  1. Reçoit tous les tokens précédents
  2. Calcule un score (logit) pour chacun des ~50 000 tokens du vocabulaire
  3. Applique softmax → distribution de probabilités
  4. Sample un token selon cette distribution
  5. Ajoute le token et recommence

Sans contrainte :
  Vocabulaire entier disponible à chaque étape → n'importe quoi possible
```

## Le token masking — l'idée centrale

```
Avec constrained decoding :

  1. LLM calcule les logits sur le vocabulaire entier
  2. Le masker identifie quels tokens sont valides à cette étape
  3. Les tokens invalides reçoivent logit = -∞ (probabilité ≈ 0)
  4. Softmax sur les tokens restants → distribution sur tokens valides
  5. Sample parmi les tokens valides uniquement

Exemple — générer {"age":
  Tokens valides à cette position :
    "1", "2", "3", ..., "9"  ← chiffres valides pour un entier
    Tokens invalides masqués :
    "abc", "true", "null", "{"  ← impossibles ici
```

## Les FSM — Finite State Machines (Outlines)

Outlines compile la contrainte en un **automate fini** avant la génération.

```
Exemple pour la regex : \d{2}-\d{2}-\d{4}  (date JJ-MM-AAAA)

États de l'automate :
  État 0 : début         → tokens valides : 0-9
  État 1 : 1er chiffre   → tokens valides : 0-9
  État 2 : 2ème chiffre  → tokens valides : "-"
  État 3 : après "-"     → tokens valides : 0-9
  État 4 : 3ème chiffre  → tokens valides : 0-9
  État 5 : après "-"     → tokens valides : 0-9
  ... (4 chiffres pour l'année)
  État final : uniquement EOS (fin de séquence)

À chaque étape de génération :
  → On regarde l'état courant de l'automate
  → On lit le masque pré-calculé pour cet état
  → On l'applique aux logits du LLM
```

## Avantage Outlines : pré-calcul des masques

```
Outlines (pré-calcul) :
  Phase de compilation (une fois) :
    Pour chaque état de l'automate →
    calculer le masque binaire sur les 50k tokens
    → coût initial : quelques secondes
  
  Phase de génération (chaque token) :
    Lookup O(1) dans la table des masques
    → coût : microsecondes par token

LM Format Enforcer (dynamique) :
  Phase de génération (chaque token) :
    Calculer dynamiquement quels tokens sont valides
    → plus flexible, mais ~O(n) par token
    → meilleur pour les contraintes dynamiques
```

## FSM pour JSON Schema

La compilation d'un JSON Schema en FSM est plus complexe.

```python
# Ce schéma Pydantic
class Produit(BaseModel):
    nom: str
    prix: float
    en_stock: bool
    tags: list[str]

# Devient une regex complexe qui garantit :
# { "nom": "<string>", "prix": <float>, "en_stock": <bool>, "tags": [<strings>] }

# Outlines transforme ce schéma en une FSM qui :
# - Garantit les accolades ouvrantes/fermantes
# - Force les guillemets autour des clés
# - Garantit les bons types pour chaque valeur
# - Respecte la syntaxe JSON stricte
```

## XGrammar — l'approche context-free grammar

Outlines construit un automate depuis les contraintes et pré-calcule les masques de tokens pour tous les états, rendant le sampling rapide mais limitant la complexité des contraintes et introduisant un coût de démarrage significatif. XGrammar utilise une approche différente avec des masques pré-calculés et du traitement parallèle pour atteindre des performances 100× supérieures.

```
Context-Free Grammar (CFG) vs Regex :

Regex :
  → Langages réguliers (type 3 dans la hiérarchie de Chomsky)
  → Ne peut pas gérer les structures imbriquées infinies
  → Exemple : JSON avec imbrication arbitraire = IMPOSSIBLE en regex pure

CFG (Grammaire Context-Free) :
  → Langages context-free (type 2)
  → Peut gérer les structures récursives et imbriquées
  → Exemple : JSON valide avec imbrication quelconque = POSSIBLE
  → Aussi : SQL, expressions arithmétiques, code source
```

## Logit processors — le point d'intégration

Dans Transformers, le constrained decoding s'intègre via les `LogitsProcessor`.

```python
from transformers import LogitsProcessor
import torch

class MonLogitsProcessor(LogitsProcessor):
    def __call__(
        self,
        input_ids: torch.LongTensor,   # tokens générés jusqu'ici
        scores: torch.FloatTensor      # logits du LLM sur tout le vocabulaire
    ) -> torch.FloatTensor:
        # Masquer les tokens invalides
        masque = calculer_masque_valide(input_ids)   # ta logique de contrainte
        scores[~masque] = float("-inf")              # -inf → probabilité ~0
        return scores

# Les librairies (Outlines, LM Format Enforcer) implémentent
# ce LogitsProcessor en interne — tu n'as pas à le faire manuellement
```

## Résumé visuel du mécanisme

```
Contrainte (regex / JSON / grammaire)
          ↓
    [Compilation]
          ↓
    Automate FSM ← Outlines, XGrammar
    ou
    Logique dynamique ← LM Format Enforcer
          ↓
    [Génération token par token]
          ↓
    LLM calcule logits (50k valeurs)
          ↓
    Masque appliqué (invalides → -∞)
          ↓
    Sampling parmi les tokens valides
          ↓
    Token choisi TOUJOURS conforme
          ↓
    Recommencer jusqu'à EOS
```

> [!tip] Compilation = startup cost
> Avec Outlines, la première génération avec un nouveau schéma prend quelques secondes (compilation de la FSM). Les générations suivantes avec le même schéma sont quasi-instantanées. En production, compiles une fois au démarrage, pas à chaque requête.

> [!warning] Le constrained decoding peut dégrader légèrement la qualité
> En restreignant les tokens disponibles, on peut forcer le modèle dans des chemins sous-optimaux pour le sens. C'est le trade-off qualité/structure. Pour les structures simples (JSON basique), l'impact est minimal. Pour les grammaires très restrictives, surveille la qualité du contenu.
