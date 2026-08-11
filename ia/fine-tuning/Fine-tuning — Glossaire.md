#ia #fine-tuning #glossaire #référence

| Terme | Définition |
|---|---|
| **Fine-tuning** | Entraînement supplémentaire d'un modèle de fondation existant sur un dataset spécifique pour le spécialiser. |
| **Modèle de fondation** | LLM pré-entraîné sur des milliards de tokens (GPT-4, Claude, LLaMA...) servant de base au fine-tuning. |
| **SFT (Supervised Fine-Tuning)** | Type de fine-tuning où le modèle apprend à partir de paires (entrée, sortie idéale). Le plus courant. |
| **RLHF** | Reinforcement Learning from Human Feedback. Fine-tuning basé sur des préférences humaines entre plusieurs réponses. |
| **RLAIF** | Reinforcement Learning from AI Feedback. Variante de RLHF où le feedback humain est remplacé par un LLM évaluateur. |
| **DPO (Direct Preference Optimization)** | Alternative plus simple à RLHF qui optimise directement le modèle sur des paires (réponse choisie, réponse rejetée). |
| **LoRA (Low-Rank Adaptation)** | Technique qui n'entraîne que de petites matrices d'adaptation (0.1-3% des paramètres) au lieu de tous les poids. |
| **QLoRA** | LoRA appliqué à un modèle quantifié en 4-bit, permettant le fine-tuning de très grands modèles sur un seul GPU. |
| **Rank (r)** | Hyperparamètre LoRA contrôlant la taille des matrices d'adaptation. Plus grand = plus expressif, plus coûteux. |
| **Full fine-tuning** | Modification de tous les poids du modèle pendant l'entraînement. Maximum de qualité, maximum de ressources. |
| **Dataset JSONL** | Format standard des données d'entraînement : un exemple JSON par ligne, avec messages system/user/assistant. |
| **Training loss** | Mesure de l'erreur du modèle sur les données d'entraînement. Doit baisser progressivement. |
| **Validation loss** | Mesure de l'erreur sur des données non vues pendant l'entraînement. Indicateur de généralisation. |
| **Overfitting** | Le modèle mémorise le dataset d'entraînement sans généraliser. Détectable quand la validation loss remonte. |
| **Catastrophic forgetting** | Phénomène où le fine-tuning fait "oublier" au modèle ses capacités générales acquises pendant le pré-entraînement. |
| **Epoch** | Un passage complet sur tout le dataset d'entraînement. Le fine-tuning fait généralement 1-5 epochs. |
| **Learning rate** | Vitesse d'ajustement des poids à chaque étape. Trop élevé = instabilité. Trop faible = entraînement lent. |
| **Gradient descent** | Algorithme d'optimisation qui ajuste les poids dans la direction qui réduit l'erreur. |
| **PPO (Proximal Policy Optimization)** | Algorithme de reinforcement learning utilisé dans RLHF pour mettre à jour le modèle de manière stable. |
| **Reward Model** | Modèle entraîné sur des préférences humaines pour attribuer un score de qualité à une réponse. Utilisé dans RLHF. |
| **Perplexité** | Mesure de confiance du modèle. Basse = modèle confiant et cohérent. Haute = modèle qui hésite. |
| **Quantification (4-bit)** | Compression des poids du modèle de float32 à int4 pour réduire la mémoire GPU nécessaire. |
| **VRAM** | Mémoire vidéo des GPU. Ressource principale limitante pour le fine-tuning. |
| **Adaptateur LoRA** | Les petites matrices A et B entraînées par LoRA. Fichier de quelques MB séparable du modèle de base. |
| **Red teaming** | Processus de test offensif pour identifier les failles et comportements non souhaités d'un modèle fine-tuné. |
| **Early stopping** | Arrêt de l'entraînement quand la validation loss remonte, pour éviter l'overfitting. |
| **Unsloth** | Librairie Python qui optimise le fine-tuning LoRA/QLoRA pour utiliser moins de mémoire et aller plus vite. |
| **vLLM** | Moteur d'inférence haute performance pour déployer des LLM open-source avec un throughput optimisé. |
| **Fuite de données d'entraînement** | Risque qu'un modèle fine-tuné restitue mot pour mot un exemple de son dataset — critique si ce dataset contient des données sensibles. |
| **Reproductibilité (fine-tuning)** | Capacité à rejouer un résultat d'entraînement identique, garantie par le versionnage du dataset, du seed aléatoire et des hyperparamètres utilisés pour chaque run. |
