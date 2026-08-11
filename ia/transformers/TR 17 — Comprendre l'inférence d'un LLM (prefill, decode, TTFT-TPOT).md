#ia #transformers #production #fondamentaux

## Inférence vs entraînement : une différence économique autant que technique

L'entraînement (voir [[IA 03 — Comment une IA apprend]]) est une opération unique, extrêmement coûteuse, qui mobilise des grappes de GPU pendant des semaines pour produire un jeu de poids figé. L'**inférence** rejoue ces poids à chaque requête : chaque exécution est courte, mais elle se répète des millions de fois et constitue l'essentiel du coût d'exploitation d'un modèle en production.

> [!info] Les outils d'inférence n'entraînent jamais de modèle
> Ollama (voir [[Ollama — Index des fiches]]), llama.cpp, vLLM et SGLang exécutent un modèle déjà entraîné — ils n'en créent aucun. Confondre les deux usages mène à mal dimensionner le matériel : l'inférence d'un modèle de 7B tient sur un GPU grand public, l'entraîner ne tient pas.

## Deux phases bien distinctes : prefill et decode

Le **prefill** lit l'intégralité du prompt d'entrée d'un seul coup. Le modèle a accès à tout le texte dès le départ, donc il peut traiter tous les tokens **en parallèle** — une opération massivement parallèle, donc rapide, même pour un prompt long.

Le **decode** génère la réponse un token à la fois. Chaque nouveau token dépend de tous les tokens précédents, y compris ceux que le modèle vient lui-même de produire — impossible de générer le 10ᵉ mot avant d'avoir écrit le 9ᵉ. Le decode est donc **séquentiel**, et c'est l'étape lente de l'inférence.

> [!warning] La vitesse ressentie dépend surtout du decode
> Un modèle peut « avaler » un prompt de 5 000 mots en une fraction de seconde (prefill parallèle), puis mettre plusieurs secondes à rédiger sa réponse (decode séquentiel) — cette asymétrie explique pourquoi le decode concentre l'essentiel des efforts d'optimisation des serveurs d'inférence (voir [[TR 16 — Servir un LLM à plusieurs utilisateurs (batching, PagedAttention, RadixAttention)]]).

## Le KV cache : éviter de tout recalculer à chaque token

Pendant le decode, produire chaque nouveau token exige d'examiner tous les tokens précédents via le mécanisme d'attention (voir [[TR 01 — L'architecture Transformer & le mécanisme d'attention]]). Sans optimisation, le modèle recalculerait à chaque étape l'analyse de l'ensemble du texte déjà traité — un gaspillage qui empire à mesure que la réponse s'allonge.

Le **KV cache** (cache des clés et valeurs, les vecteurs *Key* et *Value* du mécanisme d'attention) résout ce problème : à chaque token traité, le modèle range dans ce cache les calculs intermédiaires le concernant, et les réutilise pour le token suivant au lieu de les refaire.

> [!warning] Un contexte plus long n'est jamais gratuit
> Le KV cache grandit avec la longueur du texte traité — plus la conversation est longue, plus il occupe de mémoire. Sur un GPU partagé entre plusieurs utilisateurs, c'est le KV cache qui détermine combien de conversations simultanées la carte peut tenir : doubler la fenêtre de contexte d'un serveur peut diviser par deux le nombre d'utilisateurs acceptés en parallèle.

## Latence et débit : deux mesures, deux objectifs

La **latence** est le temps que met une seule requête à être traitée du début à la fin — ce que ressent un utilisateur isolé. Le **débit** (*throughput*) est la quantité totale de travail accompli par unité de temps, tous utilisateurs confondus — ce qui intéresse l'exploitant du service.

> [!warning] Optimiser l'un peut dégrader l'autre
> Un serveur qui regroupe beaucoup de requêtes pour maximiser le débit fait parfois attendre chaque requête un peu plus longtemps, augmentant la latence individuelle — c'est exactement le compromis détaillé dans [[TR 16 — Servir un LLM à plusieurs utilisateurs (batching, PagedAttention, RadixAttention)]].

## TTFT et TPOT : ce que ressent concrètement l'utilisateur

| Mesure | Phase concernée | Ce que l'utilisateur perçoit |
|--------|--------------------|---------------------------------|
| **TTFT** (*Time To First Token*) | Prefill | Le délai avant que « ça commence » — le blanc après avoir appuyé sur Entrée |
| **TPOT** (*Time Per Output Token*) | Decode | La vitesse à laquelle le texte « se déroule » une fois lancé |

Un TPOT de 30 millisecondes correspond à environ 33 tokens par seconde — suffisamment fluide pour une lecture humaine. Un long prompt pèse surtout sur le TTFT ; une longue réponse se ressent surtout via le TPOT.

## Pourquoi le GPU domine l'inférence

L'inférence consiste, à chaque token, en une avalanche de multiplications de matrices — des milliers de calculs identiques et indépendants, donc massivement parallélisables. Un GPU possède des milliers de petites unités de calcul travaillant de front ; un CPU dispose de quelques cœurs très polyvalents, mais bien moins nombreux.

> [!tip] Le CPU reste pertinent à petite échelle
> Pour un petit modèle, un usage mono-utilisateur ou du prototypage local, un CPU moderne suffit — c'est tout l'intérêt d'outils comme llama.cpp ou Ollama (voir [[Ollama 01 — Prérequis matériels]]). Dès qu'il s'agit d'un gros modèle ou d'un service partagé entre plusieurs utilisateurs, le GPU devient incontournable.

## Idées fausses fréquentes

| Idée reçue | Réalité |
|--------------|---------|
| « Un modèle plus gros répond plus vite » | Faux — un modèle plus gros est généralement **plus lent** en inférence, chaque token demandant plus de calculs. Souvent meilleur en qualité, pas en vitesse. |
| « Tokens par seconde, c'est la seule métrique qui compte » | Faux — un excellent débit avec un mauvais TTFT donne un service qui paraît lent à démarrer. Les deux comptent. |
| « Le CPU, c'est juste un GPU plus lent » | Faux — la différence est structurelle, pas une question de degré : le GPU est conçu pour un type de calcul que le CPU traite mal. |
| « Allonger le contexte est gratuit » | Faux — un contexte plus long gonfle le KV cache, consomme de la mémoire et réduit le nombre d'utilisateurs simultanés. |

## Pour aller plus loin

Ces fondamentaux posés, la façon dont un serveur d'inférence sert plusieurs utilisateurs à la fois (batching, PagedAttention, RadixAttention) est détaillée dans [[TR 16 — Servir un LLM à plusieurs utilisateurs (batching, PagedAttention, RadixAttention)]].

Sources : [Comprendre l'inférence d'un LLM — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/inference-llm/)
