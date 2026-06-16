#ia #constrained-decoding #bases #définition

## Qu'est-ce que le Constrained Decoding ?

Le constrained decoding (ou guided decoding / structured generation) est une technique qui **contraint le LLM à ne générer que des tokens valides** selon une structure prédéfinie — JSON, regex, grammaire — directement pendant la génération, token par token.

## Le problème fondamental

```
LLM sans contrainte :
  Prompt : "Réponds en JSON avec les champs nom et âge."
  Output : "Bien sûr ! Voici le JSON demandé :
            ```json
            { 'nom': 'Alice', 'age': trente }
            ```
            J'espère que cela répond à votre question !"

Problèmes :
  → Markdown autour du JSON (backticks)
  → Guillemets simples au lieu de doubles
  → "trente" au lieu de 30
  → Texte superflu avant et après
  → json.loads() → crash garanti
```

## La solution — masquer les tokens invalides

```
À chaque étape de génération :

  1. Le modèle calcule une distribution sur tout le vocabulaire
     (50k+ tokens avec leurs probabilités)

  2. Le constrained decoder masque les tokens invalides
     (met leur probabilité à 0)

  3. Le modèle sample parmi les tokens restants (tous valides)

  4. Le token choisi est toujours conforme à la structure

Résultat : output structurellement garanti à 100%
```

## Comparaison — avant et après

| Approche | Fiabilité | Coût | Complexité |
|---|---|---|---|
| **Prompt + espoir** | ~70% | Faible | Faible |
| **Post-processing / retry** | ~90% | Moyen (retries) | Moyenne |
| **with_structured_output()** | ~95% (API) | Faible | Faible |
| **Constrained Decoding** | **~100%** | Faible | Moyenne |

## Les 4 types de contraintes

```
1. Types de base
   → int, float, bool, date
   → "La réponse est exactement un entier"

2. Choix (choice)
   → Literal["A", "B", "C"]
   → "Le token généré est obligatoirement A, B ou C"

3. Regex
   → r"\d{4}-\d{2}-\d{2}"
   → "Génère exactement ce pattern"

4. JSON Schema / Pydantic
   → {"name": str, "age": int, "tags": list[str]}
   → "Génère un JSON valide conforme à ce schéma"

5. Grammaire (CFG)
   → EBNF : expression := term ('+' term)*
   → "Génère du texte conforme à cette grammaire formelle"
```

## Quand l'utiliser

```
✅ Utiliser quand :
  - Tu as besoin d'un JSON valide à 100% (pipeline, API, DB)
  - Tu extrais des données structurées de textes non structurés
  - Tu génères du code dans un langage spécifique
  - Tu as un format de sortie critique (médical, juridique, financier)
  - Tu utilises un modèle local (transformers, vLLM)

⚠️ Moins nécessaire quand :
  - Tu utilises Claude/GPT-4 avec with_structured_output() → déjà très fiable
  - Le format n'est pas critique (réponse conversationnelle)
  - Tu n'as pas accès au modèle local (APIs cloud)
```

## Les librairies disponibles en 2025

```
Outlines         → FSM, le plus polyvalent, intégration vLLM native
LM Format Enforcer → filtrage token par token, très flexible
XGrammar         → 100× plus rapide via masques pré-calculés, production
Guidance         → DSL Python pour contrôler la génération, Microsoft
llguidance       → moteur Guidance, pas de startup cost, très rapide
```

> [!tip] Constrained Decoding ≠ Prompting structuré
> Le prompting structuré dit au LLM "essaie de faire du JSON". Le constrained decoding **garantit** du JSON valide en rendant physiquement impossible la génération de tokens invalides. Ce n'est pas une suggestion, c'est une contrainte mathématique.

> [!info] Support natif dans vLLM
> vLLM supporte nativement le constrained decoding via Outlines, LM Format Enforcer ou XGrammar comme backends. C'est l'approche recommandée en production pour les modèles locaux.
