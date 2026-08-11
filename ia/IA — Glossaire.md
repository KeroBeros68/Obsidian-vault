#ia #glossaire #référence

| Terme | Définition |
|---|---|
| **IA (Intelligence Artificielle)** | Capacité d'une machine à simuler des comportements humains : comprendre, apprendre, raisonner, décider. |
| **Machine Learning (ML)** | Sous-domaine de l'IA où la machine apprend à partir de données sans qu'on lui programme les règles explicitement. |
| **Deep Learning** | Sous-domaine du ML utilisant des réseaux de neurones artificiels à plusieurs couches pour traiter des données complexes. |
| **LLM** | Large Language Model. Modèle entraîné sur d'énormes quantités de texte, capable de comprendre et générer du langage naturel. Ex : Claude, GPT-4. |
| **IA Générative** | IA capable de créer du nouveau contenu (texte, image, audio, vidéo) à partir d'une instruction. |
| **Prompt** | L'instruction ou le message que tu envoies à l'IA pour obtenir une réponse. |
| **Prompting** | L'art de formuler des prompts efficaces pour obtenir les meilleurs résultats d'une IA. |
| **Token** | Unité de traitement du texte par un LLM. Environ 1 token = ¾ d'un mot en anglais. Les modèles ont une limite de tokens par conversation. |
| **Hallucination** | Phénomène où un LLM invente des faits, des sources ou des citations avec une confiance totale. À toujours vérifier. |
| **Paramètres** | Les millions ou milliards de valeurs numériques internes d'un modèle, ajustées pendant l'entraînement. Ils "contiennent" le savoir du modèle. |
| **Entraînement** | Processus par lequel un modèle ajuste ses paramètres sur des données pour minimiser ses erreurs. |
| **Fine-tuning** | Entraînement supplémentaire d'un modèle existant sur des données spécifiques pour le spécialiser dans une tâche. |
| **Contexte (context window)** | La quantité de texte (en tokens) que le modèle peut "voir" et considérer en même temps dans une conversation. |
| **Apprentissage supervisé** | Type de ML où le modèle apprend à partir de données étiquetées (exemple : email = spam ou non-spam). |
| **Apprentissage non supervisé** | Type de ML où le modèle trouve lui-même des structures dans des données sans étiquettes. |
| **Apprentissage par renforcement** | Type de ML où le modèle apprend par essai/erreur en recevant des récompenses ou des pénalités. |
| **Zero-shot prompting** | Demander une réponse sans fournir d'exemple — le choix par défaut pour une tâche simple. |
| **One-shot prompting** | Fournir un seul exemple dans le prompt pour cadrer le format ou le ton attendu. |
| **Few-shot prompting** | Technique consistant à fournir 2-3 exemples dans le prompt pour que l'IA reproduise le pattern souhaité. |
| **Chain of Thought** | Technique demandant à l'IA de raisonner étape par étape avant de répondre, pour améliorer la précision. Formalisée par Wei et al. (2022). |
| **Self-Consistency** | Générer plusieurs réponses (Chain of Thought à température non nulle) et retenir la plus fréquente par vote majoritaire — plus fiable qu'une seule réponse, mais coûte N fois plus cher. |
| **API** | Interface permettant à des développeurs d'intégrer les capacités d'un modèle IA dans leurs propres applications. |
| **Temperature** | Paramètre contrôlant le degré de créativité vs précision de la réponse. Basse = déterministe, haute = créatif. |
