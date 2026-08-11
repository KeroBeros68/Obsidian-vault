#ia #ollama #sécurité #avancé

## Le comportement par défaut : local uniquement

Par défaut, Ollama n'écoute que sur `127.0.0.1:11434` — son API n'est donc accessible que depuis la machine elle-même, jamais depuis le réseau.

```bash
curl http://127.0.0.1:11434/api/tags   # Fonctionne, en local
```

## OLLAMA_HOST : ouvrir l'accès réseau sans authentification

```bash
OLLAMA_HOST=0.0.0.0 ollama serve
```

> [!warning] Aucune authentification native sur l'API Ollama
> Définir `OLLAMA_HOST=0.0.0.0` rend l'API accessible depuis n'importe quel poste du réseau — **sans aucun mécanisme d'authentification intégré**. N'importe qui sur ce réseau peut alors lister les modèles installés, en télécharger de nouveaux, en supprimer, et consommer le GPU de la machine pour ses propres requêtes.

## Sécuriser un accès distant

Si l'accès depuis un autre poste est réellement nécessaire, deux approches :

```
Option 1 : Reverse proxy authentifié
  Client → Reverse proxy (auth, TLS) → Ollama (127.0.0.1:11434)

Option 2 : Restriction par pare-feu
  Autoriser le port 11434 uniquement depuis les IP du réseau local de confiance
```

> [!tip] Ne jamais exposer 11434 directement sur Internet
> Un reverse proxy (Nginx, Caddy — voir [[Nginx 14 — Sécurisation avancée]] ou [[Caddy — Index des fiches]]) ajoute l'authentification et le TLS qu'Ollama ne fournit pas nativement. Sans cette couche, exposer directement le port revient à laisser un accès total et anonyme au moteur d'inférence et à ses modèles.

## Autres paramètres de configuration courants

```bash
OLLAMA_MODELS=/chemin/personnalise   ollama serve   # Emplacement de stockage des modèles
OLLAMA_MAX_LOADED_MODELS=2           ollama serve   # Limite de modèles chargés simultanément en mémoire
```

> [!info] Ces réglages passent par des variables d'environnement, pas un fichier de configuration
> Contrairement à des serveurs comme Redis ou Nginx (voir [[Redis 15 — Le fichier redis.conf & CONFIG REWRITE]] pour un exemple de mécanisme équivalent ailleurs dans ce vault), Ollama se configure principalement via des variables d'environnement passées au service, sans fichier de configuration central à éditer.

## Pour aller plus loin

Cela conclut le module Ollama installation & CLI — voir [[Ollama — Index des fiches]]. L'intégration Python (API REST, SDK, sorties structurées, tool calling, modèles de vision) et le dépannage approfondi font l'objet de guides distincts, non encore couverts dans ce vault — voir [[Manques]].

Sources : [Installer et utiliser Ollama — Stéphane Robert](https://blog.stephane-robert.info/docs/developper/programmation/python/ollama/)
