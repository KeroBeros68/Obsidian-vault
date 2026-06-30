#ia #bases #types

## Les types d'IA

Il existe plusieurs grandes familles d'IA, souvent imbriquées les unes dans les autres.

## 1. IA Symbolique

Basée sur des règles logiques écrites manuellement par des humains.

```
SI commande > 100€ ALORS livraison_gratuite = vrai
SI température > 38 ET toux = vrai ALORS alerte_medecin()
```

- ✅ Transparent, prévisible, facile à auditer
- ❌ Rigide, ne gère pas l'ambiguïté, ne s'améliore pas seule

## 2. Machine Learning (ML) — Apprentissage automatique

La machine apprend à partir d'exemples, sans qu'on lui écrive les règles.

| Type | Principe | Exemple concret |
|---|---|---|
| **Supervisé** | Apprend avec des données étiquetées | Détecter les spams (email = spam / pas spam) |
| **Non supervisé** | Trouve des structures seul | Regrouper des clients par comportement |
| **Par renforcement** | Apprend par essai/erreur avec récompenses | AlphaGo, robots, voitures autonomes |

> [!info] Analogie supervisé vs non supervisé
> Supervisé = apprendre avec un prof qui corrige. Non supervisé = trier des objets seul sans consigne.

## 3. Deep Learning (apprentissage profond)

Sous-catégorie du ML utilisant des **réseaux de neurones artificiels** inspirés du cerveau.

```
Données brutes
    ↓
[Couche 1] détecte des formes simples (bords, sons)
    ↓
[Couche 2] combine en formes complexes (visages, mots)
    ↓
[Couche N] produit une compréhension abstraite
    ↓
Résultat (classification, prédiction, génération)
```

- ✅ Très puissant sur des données complexes (images, texte, audio)
- ❌ Nécessite énormément de données et de puissance de calcul

## 4. IA Générative

Crée du **nouveau contenu** à partir de ce qu'elle a appris.

| Type           | Outils                  | Produit              |
| -------------- | ----------------------- | -------------------- |
| LLM            | Claude, ChatGPT, Gemini | Texte, code, analyse |
| Diffusion      | Midjourney, DALL·E      | Images               |
| Text-to-speech | ElevenLabs              | Voix réaliste        |
| Text-to-video  | Runway, Sora            | Vidéo                |
| Text-to-music  | Suno                    | Musique complète     |

> [!warning] Hallucination
> Les LLM peuvent inventer des faits avec une confiance totale. Toujours vérifier les informations importantes.

> [!tip] Mémo
> IA Symbolique → règles manuelles. ML → apprend des données. Deep Learning → réseaux de neurones. Générative → crée du contenu.
