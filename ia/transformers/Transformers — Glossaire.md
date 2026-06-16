#ia #transformers #glossaire #référence

| Terme | Définition |
|---|---|
| **Transformers** | Librairie Python HuggingFace donnant accès à des milliers de modèles pré-entraînés pour NLP, vision, audio. |
| **Hugging Face Hub** | Dépôt de 500k+ modèles open-source. Héberge les poids, configurations et datasets. |
| **pipeline()** | API haut niveau de Transformers pour l'inférence rapide. Gère automatiquement tokenisation, modèle et décodage. |
| **AutoTokenizer** | Classe qui détecte et charge automatiquement le bon tokenizer pour un modèle donné. |
| **AutoModel** | Classe générique de chargement de modèle. `AutoModelForCausalLM` pour la génération, `AutoModelForSequenceClassification` pour la classification, etc. |
| **Tokenisation** | Transformation du texte en séquences de token IDs numériques que le modèle peut traiter. |
| **BPE** | Byte Pair Encoding. Algorithme de tokenisation qui fusionne les paires de bytes les plus fréquentes. Utilisé par GPT, Mistral, LLaMA. |
| **WordPiece** | Algorithme de tokenisation utilisé par BERT. Les sous-mots de continuation commencent par `##`. |
| **SentencePiece** | Algorithme de tokenisation indépendant de la langue. Les débuts de mots commencent par `▁`. Utilisé par T5, ALBERT. |
| **Chat template** | Format standardisé pour structurer les conversations multi-tours. Appliqué via `tokenizer.apply_chat_template()`. |
| **Greedy decoding** | Stratégie de génération qui choisit toujours le token le plus probable. Déterministe mais répétitif. |
| **Beam search** | Génération qui explore plusieurs chemins simultanément. Meilleure qualité, plus lente. Paramètre : `num_beams`. |
| **Temperature** | Contrôle la créativité de la génération. 0=déterministe, 1=standard, >1=très aléatoire. |
| **Top-k sampling** | Limite le choix aux k tokens les plus probables avant de sampler. |
| **Top-p (nucleus)** | Limite le choix aux tokens représentant p% de la probabilité cumulée. |
| **TextStreamer** | Classe Transformers qui affiche chaque token généré immédiatement (streaming en console). |
| **TextIteratorStreamer** | Version itérable du TextStreamer pour les APIs asynchrones (FastAPI, etc.). |
| **GenerationConfig** | Objet de configuration réutilisable pour tous les paramètres de génération. |
| **Sentence Transformers** | Librairie spécialisée dans les embeddings sémantiques. Utilise `transformers` sous le capot. |
| **MTEB** | Massive Text Embedding Benchmark. Classement de référence pour comparer les modèles d'embedding. |
| **PEFT** | Parameter-Efficient Fine-Tuning. Librairie HuggingFace implémentant LoRA, QLoRA et autres méthodes efficaces. |
| **LoRA** | Low-Rank Adaptation. Fine-tuning de 0.1-3% des paramètres via matrices de bas rang. Paramètres : `r`, `lora_alpha`, `target_modules`. |
| **QLoRA** | LoRA appliqué à un modèle quantifié en 4-bit. Permet le fine-tuning de grands modèles sur GPU modeste. |
| **target_modules** | Les couches du modèle sur lesquelles appliquer LoRA. Typiquement `q_proj`, `k_proj`, `v_proj`, `o_proj`. |
| **r (LoRA rank)** | Rang des matrices d'adaptation LoRA. Plus grand = plus expressif, plus coûteux. Valeur typique : 8-32. |
| **lora_alpha** | Facteur de scaling LoRA. Généralement = r ou 2×r. |
| **merge_and_unload()** | Fusionne les adaptateurs LoRA dans le modèle de base pour l'inférence. Résultat = modèle standard plus rapide. |
| **Trainer** | API haut niveau de Transformers pour l'entraînement. Gère boucle, évaluation, checkpoints, logs. |
| **TrainingArguments** | Configuration complète de l'entraînement : epochs, lr, batch size, fp16, logging... |
| **SFTTrainer** | Trainer TRL spécialisé pour le fine-tuning supervisé de LLM. Gère automatiquement le chat template. |
| **TRL** | Transformer Reinforcement Learning. Librairie HuggingFace pour SFT, DPO, RLHF, PPO. |
| **DPO** | Direct Preference Optimization. Fine-tuning sur des paires (chosen, rejected) sans Reward Model séparé. |
| **beta (DPO)** | Régularisation KL dans DPO. Contrôle à quel point le modèle peut s'éloigner du modèle de référence. |
| **BitsAndBytes** | Librairie de quantification dynamique. Supporte 8-bit et 4-bit NF4. Nécessite CUDA/Linux. |
| **NF4** | NormalFloat4. Type de quantification 4-bit recommandé par BitsAndBytes pour le QLoRA. |
| **GPTQ** | Quantification post-entraînement calibrée sur un dataset. Meilleure qualité que BnB en inférence. |
| **AWQ** | Activation-Aware Weight Quantization. Protège les poids les plus importants. Meilleure qualité + vitesse. |
| **Flash Attention 2** | Réimplémentation CUDA du mécanisme d'attention. 2-4× plus rapide, -80% VRAM pour l'attention. |
| **torch.compile** | Compilation JIT du graphe PyTorch. +10-50% vitesse selon le mode et le modèle. |
| **KV cache** | Cache des clés et valeurs d'attention des tokens précédents. Essentiel pour la vitesse de génération. |
| **TGI** | Text Generation Inference. Serveur d'inférence optimisé de HuggingFace. API OpenAI-compatible. |
| **Continuous batching** | Technique TGI qui groupe dynamiquement les requêtes pour maximiser l'utilisation GPU. |
| **gradient_checkpointing** | Technique qui recalcule les activations pendant le backward pass pour économiser la VRAM. |
| **mixed precision** | Entraînement en fp16 ou bfloat16. Réduit la VRAM de 50%, souvent sans perte de qualité. |
