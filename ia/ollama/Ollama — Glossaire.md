#ia #ollama #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Ollama** | Logiciel open source (MIT) qui télécharge et exécute des LLM en local, via une API REST sur le port 11434. |
| **`ollama pull`** | Télécharge un modèle depuis le registre Ollama — rejouable sans risque, sert aussi à mettre à jour un modèle existant. |
| **`ollama run`** | Lance une session interactive avec un modèle, ou exécute une question ponctuelle si un texte est passé en argument. |
| **`ollama list` / `rm` / `show`** | Respectivement : lister les modèles installés, en supprimer un, afficher les détails d'un modèle (taille, paramètres, licence). |
| **Paramètres (modèle)** | Poids ajustables du réseau de neurones, appris pendant l'entraînement — le nombre de paramètres (ex. « 3B ») indique la taille du modèle. |
| **Quantization Q4_K_M** | Format de compression par défaut des modèles distribués par Ollama, réduisant la mémoire nécessaire au prix d'une légère perte de précision. |
| **VRAM** | Mémoire dédiée d'une carte graphique — le facteur limitant pour la taille de modèle utilisable quand un GPU est présent, à la place de la RAM système. |
| **Mémoire unifiée (Apple Silicon)** | Architecture des Mac M1-M4 où CPU et GPU intégré partagent la même mémoire, évitant la recopie des poids du modèle vers une VRAM séparée. |
| **`OLLAMA_HOST`** | Variable d'environnement contrôlant l'interface d'écoute de l'API Ollama — `0.0.0.0` l'ouvre au réseau, sans authentification native. |
| **Port 11434** | Port par défaut de l'API REST locale d'Ollama, lié à `127.0.0.1` (donc non accessible depuis le réseau) sauf configuration explicite. |
| **llama.cpp** | Moteur d'inférence bas niveau sur lequel s'appuie Ollama, offrant un contrôle plus fin mais une mise en route plus technique. |
