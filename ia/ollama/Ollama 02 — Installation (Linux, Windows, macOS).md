#ia #ollama #installation #fondamentaux

## Linux : une seule commande, à inspecter avant exécution

```bash
# Télécharger le script officiel, l'inspecter, puis l'exécuter
curl -fsSL https://ollama.com/install.sh -o ollama-install.sh
less ollama-install.sh
sh ollama-install.sh
```

> [!warning] Ne jamais exécuter un script distant à l'aveugle
> `curl | sh` en une seule commande est une pratique courante mais risquée : `less ollama-install.sh` permet de relire le script avant de l'exécuter — un réflexe à garder pour n'importe quel script d'installation téléchargé depuis Internet, pas seulement celui d'Ollama.

Le script détecte la distribution et installe Ollama en 1-2 minutes.

```bash
ollama --version
# ollama version 0.30.11

sudo systemctl status ollama
# Active: active (running)
```

En cas de problème :

```bash
sudo journalctl -u ollama -f     # Logs du service
sudo systemctl restart ollama    # Redémarrage
```

## Windows : installateur graphique

Télécharger `OllamaSetup.exe` depuis `ollama.com/download`, double-cliquer, accepter l'installation. Ollama tourne ensuite en service d'arrière-plan, signalé par une icône de lama dans la zone de notification — fermer cette icône coupe Ollama.

```powershell
ollama --version
```

> [!warning] Windows Defender scanne le premier téléchargement de chaque modèle
> Un modèle de 2 Go fraîchement téléchargé est analysé par l'antivirus intégré avant d'être utilisable, ce qui peut donner l'impression que le téléchargement est bloqué alors qu'il ne l'est pas. Ce scan n'a lieu qu'une fois par modèle ; les lancements suivants sont immédiats.

## macOS : glisser-déposer

Télécharger `Ollama-darwin.zip` depuis `ollama.com/download`, décompresser, glisser `Ollama.app` dans le dossier Applications, lancer l'application (autoriser l'ouverture si macOS le demande).

```bash
ollama --version
```

> [!info] Le GPU Apple Silicon est exploité automatiquement
> Sur M1 à M4, Ollama utilise le GPU intégré sans pilote ni configuration — voir [[Ollama 01 — Prérequis matériels]] pour l'explication de la mémoire unifiée qui rend ces machines particulièrement efficaces malgré l'absence de GPU dédié.

## Pour aller plus loin

Ollama installé, l'étape suivante est de télécharger un premier modèle et d'apprendre à gérer plusieurs modèles en parallèle — voir [[Ollama 03 — Télécharger et gérer des modèles]].

Sources : [Installer et utiliser Ollama — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/ollama/)
