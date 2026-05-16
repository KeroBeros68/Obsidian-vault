#ia #fine-tuning #données #dataset #pratique

## Les données d'entraînement

La qualité du dataset est le facteur n°1 de succès d'un fine-tuning. Un mauvais dataset produit un mauvais modèle, quel que soit le budget investi.

## Le principe fondamental

```
Qualité du modèle fine-tuné ∝ Qualité du dataset

10 exemples parfaits > 1000 exemples médiocres

Le modèle apprend à imiter exactement ce que tu lui montres.
Si tes exemples sont incohérents, biaisés ou incorrects,
le modèle le sera aussi.
```

## Format standard — JSONL (JSON Lines)

Le format universel pour les datasets de fine-tuning.

```jsonl
{"messages": [{"role": "system", "content": "Tu es un assistant support Acme Corp."}, {"role": "user", "content": "Comment retourner un produit ?"}, {"role": "assistant", "content": "Pour retourner un produit Acme, rendez-vous sur votre espace client > Mes commandes > Retourner. Vous avez 30 jours après réception. Un email de confirmation vous sera envoyé sous 24h."}]}
{"messages": [{"role": "system", "content": "Tu es un assistant support Acme Corp."}, {"role": "user", "content": "Mon colis n'est pas arrivé"}, {"role": "assistant", "content": "Je suis désolé pour ce désagrément. Votre colis est peut-être en cours d'acheminement. Pouvez-vous me communiquer votre numéro de commande pour que je vérifie le suivi en temps réel ?"}]}
```

Chaque ligne = un exemple d'entraînement complet (system + user + assistant).

## Combien d'exemples faut-il ?

```
Quantité minimale par cas d'usage :

Adapter le style/ton          : 50-200 exemples
Format de sortie structuré    : 100-500 exemples
Domaine spécialisé simple     : 200-1000 exemples
Domaine très complexe         : 1000-10 000 exemples
Comportement très spécifique  : 500-2000 exemples

Règle pratique : commencer avec 100 exemples de haute qualité.
Évaluer. Doubler si les résultats sont insuffisants.
```

> [!tip] La loi des rendements décroissants
> Passer de 0 à 100 exemples = amélioration massive. De 100 à 1000 = amélioration notable. De 1000 à 10 000 = amélioration marginale. Investis d'abord dans la qualité, pas la quantité.

## Les 5 critères de qualité d'un dataset

### 1. Cohérence
```
❌ Incohérent :
  Exemple 1 : réponse formelle ("Bonjour Madame, je vous informe...")
  Exemple 2 : réponse décontractée ("Salut ! Pas de souci !")
  → Le modèle apprendra un style aléatoire

✅ Cohérent :
  Tous les exemples respectent le même ton, style, niveau de langue
```

### 2. Représentativité
```
Le dataset doit couvrir la vraie distribution des requêtes
en production.

Si 80% des vraies requêtes concernent les retours produits,
80% du dataset doit concerner les retours produits.

❌ Dataset biaisé → modèle excellent sur les cas représentés,
                    mauvais sur les cas oubliés
```

### 3. Couverture des cas limites
```
Inclure explicitement :
  - Questions ambiguës
  - Requêtes hors périmètre (avec refus appropriés)
  - Questions en langue étrangère (si pertinent)
  - Formulations inhabituelles ou avec des fautes
  - Cas adversariaux (tentatives de manipulation)
```

### 4. Exactitude factuelle
```
Chaque réponse dans le dataset doit être :
  - Factuellement correcte
  - Vérifiée par un expert du domaine
  - À jour (pas d'informations obsolètes)

❌ Une seule erreur factuelle dans le dataset → le modèle la reproduit
   systématiquement
```

### 5. Diversité
```
Varier les formulations pour la même intention :
  "Comment retourner un produit ?"
  "Je veux renvoyer ma commande"
  "Procédure de retour ?"
  "J'ai reçu un article défectueux"
  → Toutes pointent vers le même processus
  → La diversité améliore la généralisation
```

## Sources de données

### Source 1 — Données existantes (la meilleure option)
```
Conversations de support client historiques
Tickets résolus avec leur réponse
Emails envoyés par ton équipe
Documents internes rédigés par des experts
→ Données réelles = distribution réelle
```

### Source 2 — Génération par LLM (rapide)
```python
# Générer des exemples avec un LLM puissant
prompt_génération = """
Génère 10 paires (question, réponse idéale) pour un assistant support
d'une boutique e-commerce vendant des équipements sportifs.

Format JSONL strict :
{"messages": [{"role": "system", "content": "..."}, 
               {"role": "user", "content": "..."}, 
               {"role": "assistant", "content": "..."}]}

Questions variées : retours, livraisons, tailles, paiement, garanties.
Ton : professionnel et chaleureux. Réponses : concises et actionables.
"""

exemples_générés = llm_puissant.invoke(prompt_génération)
```

> [!warning] Toujours faire relire les données générées par un LLM
> Les LLM peuvent générer des exemples incohérents, factuellement incorrects ou ne correspondant pas à ton style. Une revue humaine est indispensable.

### Source 3 — Annotation humaine (la plus chère, la plus fiable)
```
Processus :
  1. Collecter des questions réelles
  2. Les soumettre à des annotateurs experts
  3. Les annotateurs rédigent la réponse idéale
  4. Revue et validation par un second annotateur
  5. Itération jusqu'à accord inter-annotateurs

Coût : $1-10 par exemple selon la complexité
Plateformes : Scale AI, Labelbox, Amazon MTurk, Argilla
```

## Nettoyage et préparation

```python
import json

def nettoyer_dataset(fichier_jsonl: str) -> list:
    exemples_propres = []
    erreurs = 0
    
    with open(fichier_jsonl) as f:
        for i, ligne in enumerate(f):
            try:
                exemple = json.loads(ligne.strip())
                
                # Vérifications de base
                assert "messages" in exemple
                assert len(exemple["messages"]) >= 2
                
                # Vérifier la longueur (éviter les exemples trop courts ou trop longs)
                réponse = exemple["messages"][-1]["content"]
                nb_mots = len(réponse.split())
                if nb_mots < 5:
                    continue  # réponse trop courte
                if nb_mots > 500:
                    continue  # réponse probablement trop longue
                
                exemples_propres.append(exemple)
                
            except Exception as e:
                erreurs += 1
                print(f"Ligne {i+1} invalide : {e}")
    
    print(f"{len(exemples_propres)} exemples valides, {erreurs} erreurs")
    return exemples_propres

# Séparation train/validation
import random
random.shuffle(exemples_propres)
seuil = int(0.9 * len(exemples_propres))
train = exemples_propres[:seuil]       # 90% pour l'entraînement
validation = exemples_propres[seuil:]  # 10% pour la validation
```

> [!warning] Sépare toujours train et validation
> Le dataset de validation ne doit jamais être vu pendant l'entraînement. C'est lui qui mesure si le modèle généralise vraiment ou s'il a juste mémorisé le training set.
