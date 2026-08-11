#ia #ollama #pièges #erreurs #debugging

## 🪤 Piège 1 — Exposer OLLAMA_HOST=0.0.0.0 sans protection

```bash
OLLAMA_HOST=0.0.0.0 ollama serve   # ❌ Accessible à tout le réseau, sans authentification
```

> [!warning] N'importe qui sur le réseau peut alors tout faire
> Lister les modèles, en télécharger, en supprimer, et consommer le GPU de la machine pour ses propres requêtes — sans qu'aucune identification ne soit demandée. Toujours passer par un reverse proxy authentifié ou restreindre l'accès par pare-feu au réseau de confiance. Voir [[Ollama 05 — Sécurité réseau (OLLAMA_HOST & port 11434)]].

---

## 🪤 Piège 2 — Choisir un modèle trop gros pour la RAM disponible

```
16 Go de RAM, tentative de lancer un modèle 70B (nécessite 64 Go+)
→ Chargement très lent ou échec, bascule sur le disque (swap)
```

> [!warning] Vérifier la RAM avant de télécharger, pas après
> Un modèle qui ne tient pas en mémoire devient inutilisable en pratique, même s'il se télécharge sans erreur. Se référer au tableau de dimensionnement en [[Ollama 01 — Prérequis matériels]] avant de choisir une taille de modèle.

---

## 🪤 Piège 3 — Confondre RAM système et VRAM avec un GPU présent

```
32 Go de RAM système, mais seulement 8 Go de VRAM sur le GPU
→ Un modèle 13B dépasse largement la VRAM, malgré une RAM système généreuse
```

> [!warning] Avec GPU, c'est la VRAM qui dimensionne, pas la RAM
> Dès qu'un modèle déborde de la VRAM vers la RAM système, la vitesse chute d'un facteur 2 à 10. Dimensionner le choix du modèle sur la VRAM disponible, pas sur la RAM totale de la machine. Voir [[Ollama 01 — Prérequis matériels]].

---

## 🪤 Piège 4 — Accumuler des modèles sans faire le ménage

```bash
ollama list
# 8 modèles installés, dont 3 jamais réutilisés depuis leur test initial
```

> [!tip] `ollama rm` régulièrement
> Un modèle 70B occupe à lui seul une quarantaine de gigaoctets. Supprimer les modèles testés une fois et non retenus évite de saturer le disque — `ollama pull` les retéléchargera si besoin plus tard, sans perte définitive.

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| `OLLAMA_HOST=0.0.0.0` exposé sans protection | Reverse proxy authentifié ou restriction pare-feu |
| Modèle trop gros pour la RAM disponible | Vérifier le tableau de dimensionnement avant de télécharger |
| RAM système confondue avec la VRAM du GPU | Dimensionner sur la VRAM quand un GPU est présent |
| Modèles accumulés sans être supprimés | `ollama rm` sur les modèles non réutilisés |
