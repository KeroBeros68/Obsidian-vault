#ia #fine-tuning #lora #technique #intermédiaire

## Full fine-tuning et LoRA

Deux approches techniques pour modifier les poids d'un modèle, avec des compromis très différents en ressources et en qualité.

## Full fine-tuning — modifier tous les poids

### Principe
```
Modèle de base (7B paramètres)
        ↓
Entraînement sur ton dataset
        ↓
Tous les 7 milliards de poids sont mis à jour
        ↓
Modèle spécialisé (7B paramètres, tous différents)
```

### Ressources nécessaires
```
Modèle 7B  (ex: Mistral 7B)  → ~4 GPU A100 (80GB VRAM chacun)
Modèle 13B (ex: LLaMA 13B)   → ~8 GPU A100
Modèle 70B (ex: LLaMA 70B)   → ~16 GPU A100

Temps d'entraînement sur 1000 exemples :
  7B  → quelques heures
  70B → plusieurs jours

Coût sur AWS/GCP :
  7B  → $50-200 par run d'entraînement
  70B → $500-2000 par run
```

### Forces et limites
```
✅ Qualité maximale — tous les poids s'adaptent
✅ Le modèle peut vraiment "désapprendre" des comportements
❌ Très coûteux en GPU et en argent
❌ Risque de catastrophic forgetting (oublie ses capacités générales)
❌ Nécessite de stocker un modèle complet par spécialisation
```

---

## LoRA — Low-Rank Adaptation

### Le problème que LoRA résout

Le full fine-tuning modifie des milliards de poids. LoRA part d'une observation mathématique :

> Les changements de poids lors du fine-tuning ont une **rang intrinsèque bas**.
> On peut les approximer avec des matrices beaucoup plus petites.

### Principe mathématique simplifié
```
Full fine-tuning :
  W (poids originaux) + ΔW (tous les changements) = W'
  ΔW a la même taille que W → très coûteux

LoRA :
  ΔW ≈ A × B  (deux petites matrices de rang r)
  A : [d × r]   r << d
  B : [r × d]
  
  Exemple pour une couche 4096×4096 avec r=8 :
  Full FT : 4096 × 4096 = 16 millions de paramètres à entraîner
  LoRA    : 4096×8 + 8×4096 = 65 536 paramètres seulement
  → 244× moins de paramètres !
```

### Visualisation

```
         Modèle original (gelé)
         ┌─────────────────────┐
Input →  │   W (4096×4096)     │  → 
         └─────────────────────┘
                  +
         Adaptateurs LoRA (entraînés)
         ┌──────┐   ┌──────┐
Input →  │  A   │ → │  B   │  → 
         │4096×8│   │8×4096│
         └──────┘   └──────┘

Sortie finale = W × input + (A × B) × input × α
```

### Hyperparamètres clés de LoRA

| Paramètre | Description | Valeurs courantes |
|---|---|---|
| **r (rank)** | Rang des matrices d'adaptation. Plus grand = plus expressif mais plus coûteux | 4, 8, 16, 32 |
| **alpha** | Facteur de scaling de LoRA (souvent = r ou 2×r) | 8, 16, 32 |
| **target_modules** | Quelles couches adapter | q_proj, v_proj, k_proj... |
| **dropout** | Régularisation pour éviter l'overfitting | 0.05 à 0.1 |

### QLoRA — LoRA + quantification 4-bit

Extension de LoRA permettant de fine-tuner des modèles énormes sur un seul GPU.

```
Modèle 70B normalement : ~140GB VRAM (impossible sur un seul GPU)

QLoRA :
  1. Quantifier le modèle en 4-bit  → ~35GB VRAM
  2. Appliquer LoRA sur la version quantifiée
  → Fine-tuning d'un 70B sur 1 GPU A100 (80GB) !
```

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model

# Configuration QLoRA
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype="bfloat16"
)

# Charger le modèle en 4-bit
modèle = AutoModelForCausalLM.from_pretrained(
    "mistralai/Mistral-7B-v0.1",
    quantization_config=bnb_config,
    device_map="auto"
)

# Configurer LoRA
lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

modèle_lora = get_peft_model(modèle, lora_config)
modèle_lora.print_trainable_parameters()
# → trainable params: 41,943,040 || all params: 3,793,069,056 || trainable%: 1.11
```

## Comparaison Full FT vs LoRA

| Critère | Full Fine-tuning | LoRA | QLoRA |
|---|---|---|---|
| **GPU requis** | Nombreux A100 | 1-2 A100 | 1 A100 |
| **Mémoire** | Très élevée | Réduite | Très réduite |
| **Paramètres entraînés** | 100% | 0.1-3% | 0.1-3% |
| **Qualité** | Maximale | Très proche | Légère dégradation |
| **Vitesse** | Lente | Rapide | Rapide |
| **Coût** | Très élevé | Moyen | Faible |
| **Stockage modèle** | Modèle complet | Base + adaptateurs | Base + adaptateurs |

> [!tip] LoRA pour 95% des cas
> Sauf besoins très spécifiques (fine-tuning massif, modification profonde du comportement), LoRA ou QLoRA donnent des résultats quasi-identiques au full fine-tuning pour une fraction du coût.

> [!info] Les adaptateurs LoRA sont légers
> Les poids LoRA entraînés font quelques MB seulement (vs des GB pour un modèle complet). Tu peux avoir plusieurs adaptateurs LoRA spécialisés pour le même modèle de base.
