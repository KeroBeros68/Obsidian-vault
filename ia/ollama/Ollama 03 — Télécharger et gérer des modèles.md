#ia #ollama #fondamentaux

## Décoder le nom d'un modèle

Un nom comme `llama3.2:3b` porte deux informations : la famille (Llama 3.2, développée par Meta) et la taille en nombre de paramètres (3 milliards). Les paramètres sont les poids ajustables du réseau de neurones — plus il y en a, plus le modèle capture de régularités du langage, mais plus il consomme de mémoire et de calcul. Un modèle plus petit n'est pas « moins bon » dans l'absolu : il est plus rapide et moins précis sur les tâches de raisonnement complexe.

| Taille du modèle | RAM minimale | Usage typique |
|---------------------|----------------|------------------|
| 1-3B | 4-8 Go | Questions simples, résumés, traduction |
| 7B | 8-16 Go | Code, rédaction, raisonnement |
| 13B | 16-32 Go | Analyse complexe, créativité |
| 70B | 64 Go+ | Recherche, usage professionnel |

## Télécharger un modèle

```bash
ollama pull llama3.2
```

```
pulling manifest
pulling 74701a8c35f6... 50% ▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░ 1.1 GB/2.0 GB 45 MB/s
```

Le téléchargement se fait morceau par morceau, avec vérification d'intégrité, puis optimisation pour la machine locale — généralement 2 à 10 minutes selon la connexion.

> [!tip] `ollama pull` est rejouable sans risque
> Si le modèle est déjà présent, la commande se contente de vérifier les couches déjà téléchargées et se termine en quelques secondes. C'est aussi la commande à utiliser pour mettre à jour un modèle vers une nouvelle version publiée sur le registre.

## Les quatre commandes de gestion

```bash
ollama list              # Lister les modèles installés localement
ollama pull mistral       # Télécharger un nouveau modèle
ollama rm mistral         # Supprimer un modèle, libérer l'espace disque
ollama show llama3.2      # Détails d'un modèle (taille, paramètres, licence)
```

> [!tip] `ollama rm` est la commande la plus utile au quotidien
> Les modèles s'accumulent vite, et un modèle 70B occupe une quarantaine de gigaoctets — faire le ménage des modèles non utilisés évite de saturer le disque inutilement.

## Comparatif des modèles courants

| Modèle | Taille | Point fort | Commande |
|--------|--------|-------------|----------|
| llama3.2 | 2 Go | Polyvalent, bon en français | `ollama pull llama3.2` |
| mistral | 4 Go | Raisonnement, logique | `ollama pull mistral` |
| codellama | 4 Go | Génération de code | `ollama pull codellama` |
| gemma | 2 Go | Léger et rapide | `ollama pull gemma` |
| phi | 1,5 Go | Ultra-léger | `ollama pull phi` |

> [!tip] Ne pas tout installer d'emblée
> Chaque modèle occupe sa propre place sur le disque. Commencer par `llama3.2` (bon compromis généraliste), ajouter `codellama` pour du développement intensif, ou `phi` sur une machine modeste — plutôt que d'installer toute la liste par précaution.

## Pour aller plus loin

Un modèle téléchargé, place à la première conversation en ligne de commande — voir [[Ollama 04 — Première conversation & CLI]].

Sources : [Installer et utiliser Ollama — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/ollama/)
