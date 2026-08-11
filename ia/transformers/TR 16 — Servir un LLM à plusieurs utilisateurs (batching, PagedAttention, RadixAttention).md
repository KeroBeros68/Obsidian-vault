#ia #transformers #production #avancé

## Un modèle, plusieurs utilisateurs : un problème différent de l'inférence seule

Faire tourner un LLM pour soi-même (voir [[TR 06 — Génération de texte]]) et le servir à des dizaines d'utilisateurs simultanément posent des problèmes différents. Traiter les requêtes une par une — l'utilisateur A attend sa réponse complète avant que le serveur ne passe à l'utilisateur B — est simple mais terriblement inefficace.

> [!info] Prérequis : prefill, decode et KV cache
> Cette fiche s'appuie directement sur la distinction prefill/decode et le fonctionnement du KV cache — si ces notions ne sont pas encore claires, [[TR 17 — Comprendre l'inférence d'un LLM (prefill, decode, TTFT-TPOT)]] les couvre en détail avant d'aborder le serving multi-utilisateurs.

## Pourquoi une requête isolée sous-utilise le GPU

La génération token par token se décompose en deux phases : le **prefill** (traiter le prompt d'entrée en une seule passe, opération qui sature le calcul) et le **decode** (générer les tokens de sortie un par un, opération qui déplace beaucoup de données — les poids du modèle — pour assez peu de calcul par token).

> [!info] Le decode est limité par la bande passante mémoire, pas par le calcul
> Pendant le decode d'une requête isolée, le GPU passe le plus clair de son temps à déplacer les poids du modèle depuis sa mémoire plutôt qu'à calculer — comme un camion de déménagement qui ferait l'aller-retour avec un seul carton. Le problème à résoudre : comment occuper pleinement le GPU alors qu'une requête seule ne lui donne pas assez de travail.

## Le batching : traiter plusieurs requêtes ensemble

Regrouper plusieurs requêtes dans un même calcul coûte à peine plus cher qu'une requête seule : les poids du modèle, déjà chargés, servent à toutes les requêtes du lot d'un coup. Le **débit** (nombre total de tokens produits par seconde) grimpe fortement.

### Batching statique vs continuous batching

Le batching statique forme un lot, lance le calcul, et attend que **toutes** les requêtes du lot soient terminées avant de constituer le lot suivant — une requête qui génère 20 tokens reste coincée avec une requête qui en génère 500, laissant des places vides pendant que de nouvelles requêtes patientent dehors.

Le **continuous batching** corrige ce gaspillage : à chaque token, le serveur retire du lot les requêtes terminées et y injecte les nouvelles en attente. Le lot se recompose en permanence, le GPU ne traite jamais de place vide. C'est la technique qu'emploient les serveurs d'inférence modernes (vLLM, SGLang, TGI — voir [[TR 15 — Déploiement avec Inference Endpoints]]).

> [!tip] L'analogie du bus
> Batching statique : un bus qui attend que tous ses passagers soient arrivés à destination avant de repartir du dépôt. Continuous batching : un bus normal — à chaque arrêt, ceux qui sont arrivés descendent, de nouveaux montent, le bus roule en continu, toujours plein.

## PagedAttention : ranger le KV cache sans gaspiller

Le batching pose un nouveau défi : chaque requête du lot a son propre KV cache (voir [[Transformers — Glossaire]]), qui grandit au fil de la génération. L'approche naïve réserve, par requête, un bloc contigu dimensionné au pire cas (la longueur de réponse maximale possible) — la plupart des réponses étant courtes, l'essentiel de cette mémoire reste inutilisé.

> [!info] La mémoire virtuelle d'un système d'exploitation, appliquée au KV cache
> PagedAttention découpe le cache en petites pages de taille fixe, allouées à la demande au fur et à mesure que la réponse s'allonge — plus de gros bloc réservé « au cas où », chaque requête ne consomme que ce qu'elle utilise vraiment. Résultat : bien plus de requêtes simultanées tiennent dans la même carte GPU.

## RadixAttention : réutiliser les débuts identiques

Beaucoup de requêtes commencent pareil : un assistant a toujours le même system prompt, un chatbot renvoie tout l'historique à chaque tour, un système RAG répète les mêmes consignes (voir [[RAG 03 — Naive RAG]] pour le prompt de génération type). Sans optimisation, le serveur recalcule à chaque fois le KV cache de cette partie commune.

> [!info] Une optimisation propre à SGLang
> RadixAttention indexe les débuts de texte déjà calculés et réutilise leur cache quand une nouvelle requête partage le même préfixe — le prefill de la partie commune devient quasi gratuit. L'effet est nul si les requêtes n'ont rien en commun, mais considérable sur des charges répétitives (agents, chatbots multi-tours, RAG).

## Répartir un gros modèle sur plusieurs GPU : le tensor parallelism

Un modèle de 70 milliards de paramètres en 16 bits (voir [[TR 13 — Quantification (BitsAndBytes, GPTQ, AWQ, GGUF)]] pour le calcul mémoire par précision) réclame environ 140 Go — bien au-delà des 80 Go d'un GPU haut de gamme. Le **tensor parallelism** découpe le modèle sur plusieurs GPU, qui calculent chacun leur part puis échangent leurs résultats ; côté utilisateur, rien ne change, un seul serveur est interrogé. Les serveurs d'inférence exposent ce réglage sous un nom comme `tensor-parallel-size`.

## Le compromis débit / latence sous charge

Toutes ces techniques visent le débit, mais avec une contrepartie : plus les lots sont remplis, meilleur est le débit total, mais chaque requête individuelle attend un peu plus car elle partage le GPU avec davantage de voisines. À l'inverse, des lots quasi vides donnent une latence minimale par requête, mais un débit médiocre et un GPU sous-exploité.

> [!warning] Il n'existe pas de réglage universellement bon
> Un chatbot interactif privilégie la latence (réponse rapide, quitte à sous-utiliser le GPU) ; un traitement par lots de documents privilégie le débit (traiter le plus de contenu possible, quitte à ce que chaque document individuel attende). Choisir et régler un serveur d'inférence, c'est avant tout placer ce curseur en connaissance de cause plutôt que de chercher une configuration par défaut qui conviendrait à tous les usages.

## Pour aller plus loin

Ce module continue avec les modalités concrètes de déploiement (Hugging Face Inference Endpoints, TGI) dans [[TR 15 — Déploiement avec Inference Endpoints]].

Sources : [Servir un LLM à plusieurs utilisateurs — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/servir-llm/)
