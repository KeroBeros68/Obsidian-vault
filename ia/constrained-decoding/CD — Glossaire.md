#ia #constrained-decoding #glossaire #référence

| Terme | Définition |
|---|---|
| **Constrained Decoding** | Technique qui restreint la génération du LLM aux seuls tokens valides selon une contrainte prédéfinie (JSON, regex, grammaire). Garantit la conformité structurelle à 100%. |
| **Guided Decoding** | Synonyme de Constrained Decoding, utilisé notamment par vLLM. |
| **Structured Generation** | Terme utilisé par Outlines. Génération de texte garantissant un format structuré précis. |
| **Token Masking** | Le mécanisme central : les tokens invalides reçoivent un logit de -∞ (probabilité ~0) à chaque étape de génération. |
| **LogitsProcessor** | Interface Transformers permettant de modifier les logits du LLM avant le sampling. Point d'intégration du constrained decoding dans Transformers. |
| **FSM (Finite State Machine)** | Automate fini utilisé par Outlines pour encoder les contraintes. Chaque état correspond à une position dans la structure et définit les tokens valides suivants. |
| **Compilation** | Phase préalable à la génération où la contrainte (regex, JSON Schema) est convertie en FSM ou masques de tokens. Coût one-time, pas par requête. |
| **Startup Cost** | Temps de compilation de la FSM ou des masques lors de la première utilisation d'un schéma avec Outlines. Peut durer quelques secondes. |
| **Outlines** | Librairie de référence pour le constrained decoding via FSM. Par dottxt-ai (ex-normal computing). Intégration native vLLM. |
| **LM Format Enforcer** | Librairie de constrained decoding via filtrage dynamique. Plus flexible qu'Outlines pour les contraintes dynamiques. |
| **XGrammar** | Moteur haute performance (~100× plus rapide que LM Format Enforcer) via masques pré-calculés et traitement parallèle. Backend recommandé en production. |
| **Guidance** | DSL Python de Microsoft qui entremêle génération et logique de contrôle. Moteur llguidance sous le capot. |
| **llguidance** | Moteur Rust sous-jacent de Guidance. Calcule les masques à la volée sans startup cost. |
| **Context-Free Grammar (CFG)** | Grammaire formelle (type 2 Chomsky) capable de décrire les langages avec structures imbriquées. Nécessaire pour le JSON arbitrairement imbriqué ou le SQL. |
| **EBNF** | Extended Backus-Naur Form. Format standard pour définir les grammaires CFG dans Outlines et XGrammar. |
| **Regex** | Expression régulière. Contrainte de type 3 (Chomsky). Suffit pour les formats simples (dates, IPs, codes postaux). |
| **JSON Schema** | Standard pour décrire la structure d'un objet JSON. Outlines, LM Format Enforcer et XGrammar le compilent en FSM ou masques. |
| **Pydantic** | Librairie Python de validation. Les modèles Pydantic servent directement de schéma pour le constrained decoding (`outlines.generate.json(model, MonPydantic)`). |
| **guided_json** | Paramètre vLLM pour spécifier un JSON Schema à respecter pendant la génération. |
| **guided_regex** | Paramètre vLLM pour spécifier une regex à respecter pendant la génération. |
| **guided_choice** | Paramètre vLLM pour forcer la sortie à être exactement l'une des valeurs d'une liste. |
| **guided_grammar** | Paramètre vLLM pour spécifier une grammaire CFG en EBNF. Nécessite le backend Outlines. |
| **guided_decoding_backend** | Paramètre vLLM pour choisir le backend : `"outlines"`, `"lm-format-enforcer"`, ou `"xgrammar"`. |
| **prefix_allowed_tokens_fn** | Paramètre de `model.generate()` dans Transformers qui permet d'injecter un constrained decoder via LM Format Enforcer. |
| **Taux d'hallucination de format** | Pourcentage de sorties ne respectant pas le format demandé. Le constrained decoding le réduit à ~0%. |
| **with_structured_output()** | Méthode LangChain/LiteLLM qui utilise le function calling des APIs cloud pour garantir une sortie structurée. Moins absolu que le constrained decoding local. |
| **FHIR** | Fast Healthcare Interoperability Resources. Format médical strict — cas d'usage typique du constrained decoding en santé. |
