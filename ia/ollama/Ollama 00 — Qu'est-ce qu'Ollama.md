#ia #ollama #fondamentaux

## Exécuter des LLM en local, sans cauchemar de configuration

Ollama est un logiciel gratuit et open source (licence MIT) qui télécharge et exécute des grands modèles de langage directement sur la machine de l'utilisateur, via une commande simple et une API locale exposée sur le port `11434`. Une fois un modèle téléchargé, tout fonctionne hors ligne — aucune donnée ne transite vers un serveur tiers.

## Le problème qu'Ollama résout

Faire tourner un LLM soi-même, sans outil dédié, implique de télécharger des poids de plusieurs gigaoctets, d'installer les bonnes versions de Python/PyTorch/CUDA, de configurer des dizaines de paramètres, puis d'écrire du code pour charger et interroger le modèle — plusieurs heures de friction même pour un développeur expérimenté. Ollama réduit tout cela à une seule commande d'installation et une commande de téléchargement par modèle.

## Cloud vs local : ce qui change concrètement

| | LLM cloud (ChatGPT, Claude...) | Ollama (local) |
|---|-------------------------------------|--------------------|
| Confidentialité | Les données transitent par des serveurs tiers | Les données restent sur la machine |
| Coût | Facturé à l'usage | Gratuit après l'achat du matériel |
| Connexion Internet | Obligatoire | Optionnelle (après téléchargement du modèle) |
| Vitesse | Dépend de la latence réseau | Dépend du matériel local |
| Disponibilité | Peut être saturé ou en maintenance | Toujours disponible |
| Personnalisation | Limitée | Totale (création de modèles personnalisés) |

> [!tip] Les cas d'usage où Ollama s'impose
> Code sous NDA ou données réglementées qui ne peuvent pas quitter la machine, budget serré, travail récurrent hors ligne, ou besoin de personnalisation totale du modèle — voir aussi [[LLMOps — Index des fiches]] pour la question plus large du déploiement d'applications LLM en production.

## Ollama n'est pas le seul moteur d'inférence local

| Moteur | Point fort | Compromis |
|--------|-------------|-----------|
| **Ollama** | Simplicité de mise en route, zéro configuration | Réglage fin limité |
| **llama.cpp** | Contrôle fin de la quantization, tourne sur du matériel modeste | Plus technique à configurer |
| **vLLM** | Débit élevé sur GPU, sert plusieurs utilisateurs simultanément | Pensé pour la production servie, pas l'usage local individuel |
| **TGI** (Text Generation Inference, Hugging Face) | Cible la production, s'intègre à l'écosystème Hugging Face | Même terrain que vLLM, complexité comparable |

> [!info] Ollama s'appuie en réalité sur llama.cpp
> Sous le capot, Ollama utilise le moteur d'inférence de llama.cpp, mais l'enrobe d'une gestion de modèles (téléchargement, versions, API REST) qui rend l'ensemble beaucoup plus simple à opérer qu'llama.cpp seul.

## Pour aller plus loin

Avant d'installer quoi que ce soit, vérifier que la machine a les ressources nécessaires (RAM, disque, GPU) évite bien des déconvenues — voir [[Ollama 01 — Prérequis matériels]].

Sources : [Installer et utiliser Ollama — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/ollama/)
