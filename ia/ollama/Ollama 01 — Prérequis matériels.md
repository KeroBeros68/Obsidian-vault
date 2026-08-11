#ia #ollama #fondamentaux

## La RAM : le facteur le plus limitant

Un modèle Ollama est entièrement chargé en mémoire au lancement. Une RAM insuffisante empêche le chargement, ou force une bascule sur le disque (swap) qui ralentit drastiquement l'inférence.

| RAM disponible | Modèles utilisables | Expérience |
|-----------------|------------------------|------------|
| 4 Go | Insuffisant | Ollama refuse de lancer la plupart des modèles |
| 8 Go | Modèles légers (3B) | Fonctionnel mais limité |
| 16 Go | Modèles moyens (7B) | Bonne expérience pour la plupart des usages |
| 32 Go | Grands modèles (13B) | Excellente expérience |
| 64 Go+ | Très grands modèles (70B) | Usage professionnel |

## L'espace disque

| Modèle | Taille sur disque |
|--------|---------------------|
| Llama 3.2 (3B) | ~2 Go |
| Mistral (7B) | ~4 Go |
| CodeLlama (7B) | ~4 Go |
| Llama 3.1 (70B) | ~40 Go |

> [!tip] Garder de la marge
> Prévoir au moins 20 Go libres pour télécharger plusieurs modèles et les comparer, sans avoir à en supprimer un avant d'en essayer un autre.

## Le GPU : optionnel, mais change tout sur la vitesse

Ollama fonctionne sur CPU seul (2-10 secondes par réponse) ou avec un GPU NVIDIA (0,5-2 secondes), détecté et utilisé automatiquement dès qu'au moins 8 Go de VRAM sont disponibles.

> [!info] Le cas particulier des Mac Apple Silicon
> Sur M1 à M4, la mémoire est **unifiée** entre CPU et GPU intégré : le modèle n'a pas besoin d'être recopié dans une VRAM séparée. Un MacBook avec 16 Go de RAM peut ainsi charger des modèles qu'un PC portable sans GPU dédié peinerait à faire tourner — sans aucun pilote ni configuration à installer.

## Dimensionner selon la VRAM plutôt que la RAM (avec GPU)

Si un GPU est présent, c'est sa VRAM qui devient le facteur limitant, pas la RAM système. En quantization `Q4_K_M` (le format par défaut des modèles distribués par Ollama), prévoir approximativement autant de VRAM que la taille du fichier téléchargé, plus 1-2 Go pour le contexte.

| VRAM disponible | Modèles utilisables |
|-------------------|------------------------|
| 8 Go | 7B-8B (Llama 3.1 8B, Qwen3 8B) |
| 16 Go | 13B-14B |
| 24 Go+ | 32B |
| Sans GPU | Rester sur 1B-3B, sinon la génération devient pénible |

> [!warning] Le seuil VRAM → RAM coûte cher en vitesse
> Dès qu'un modèle déborde de la VRAM vers la RAM système, la vitesse de génération chute d'un facteur 2 à 10. Un modèle plus petit qui tient entièrement en VRAM est presque toujours préférable à un modèle ambitieux qui déborde.

## Récapitulatif des prérequis

| Composant | Minimum | Recommandé | Idéal |
|-----------|---------|-------------|-------|
| RAM | 8 Go | 16 Go | 32 Go |
| Disque | 10 Go libres | 50 Go libres | 100 Go+ (SSD) |
| Processeur | 64-bit moderne | Multicœur récent | Apple M1+ ou Intel i7+ |
| GPU | Non requis | NVIDIA 8 Go VRAM | NVIDIA 16 Go+ VRAM |

## Pour aller plus loin

Une fois la compatibilité matérielle vérifiée, l'installation elle-même est détaillée dans [[Ollama 02 — Installation (Linux, Windows, macOS)]].

Sources : [Installer et utiliser Ollama — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/ollama/)
