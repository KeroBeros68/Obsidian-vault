#devops #docker #pièges #erreurs #debugging

## 🪤 Piège 1 — Croire que supprimer un conteneur supprime les données

```bash
docker run -d --name db postgres:17   # ❌ pas de volume déclaré
docker rm -f db                        # toutes les données sont perdues
```

> [!warning] Couche RW éphémère
> Sans volume explicite, toutes les données écrites dans le conteneur disparaissent à sa suppression. Voir [[Docker 05 — Volumes & persistance]].

---

## 🪤 Piège 2 — COPY . . trop tôt dans le Dockerfile

```dockerfile
# ❌ Casse le cache à chaque changement de code
COPY . .
RUN pip install -r requirements.txt

# ✅ Garde le cache des dépendances intact
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

> [!tip] Mémo
> Place en haut du Dockerfile ce qui change rarement (dépendances), en bas ce qui change souvent (code). Voir [[Docker 03 — Dockerfile]].

---

## 🪤 Piège 3 — Confondre EXPOSE et publication de port

```dockerfile
EXPOSE 3000   # ❌ ne rend RIEN accessible depuis l'hôte
```

```bash
docker run -p 8080:3000 myapp   # ✅ seule commande qui publie réellement
```

> [!warning] EXPOSE est documentaire
> `EXPOSE` dans un Dockerfile n'a aucun effet réseau réel. Voir [[Docker 06 — Réseaux]].

---

## 🪤 Piège 4 — Compter sur le bridge par défaut pour la résolution par nom

```bash
docker run -d --name web nginx
docker run -d --name app myapp
# app ne peut PAS résoudre "web" par son nom sur le bridge par défaut
```

> [!tip] Mémo
> Créer toujours un réseau bridge personnalisé (`docker network create`) ou utiliser Compose, qui le fait automatiquement.

---

## 🪤 Piège 5 — `docker compose down -v` en pensant garder les données

```bash
docker compose down -v   # ❌ supprime aussi les volumes nommés !
```

> [!warning] -v est destructeur
> Le flag `-v` supprime les volumes nommés déclarés dans le fichier Compose. Sans lui, `docker compose down` seul préserve les volumes.

---

## 🪤 Piège 6 — Déployer avec :latest en production

```bash
docker pull myapp:latest   # ❌ contenu imprévisible, change à chaque push
```

```bash
docker pull myapp:1.4.2    # ✅ version explicite, traçable
```

> [!warning] latest ne garantit rien
> `latest` est juste le tag par défaut si aucun n'est précisé — il ne signale ni stabilité ni version figée. Voir [[Docker 04 — Registry & distribution]].

---

## 🪤 Piège 7 — Confondre docker run et docker exec

```bash
docker run myapp bash   # ❌ crée un NOUVEAU conteneur, pas une session dans l'existant
```

```bash
docker exec -it my-container bash   # ✅ ouvre une session dans le conteneur déjà actif
```

> [!warning] run crée, exec entre
> `docker run` démarre toujours un nouveau conteneur à partir d'une image. Pour interagir avec un conteneur déjà en cours d'exécution, c'est `docker exec` qu'il faut utiliser. Voir [[Docker 02 — Cycle de vie & debugging]].

---

## 🪤 Piège 8 — Laisser le conteneur tourner en root par défaut

```dockerfile
FROM node:20-alpine
# ❌ Aucun USER déclaré : le processus tourne en root par défaut
CMD ["node", "server.js"]
```

```dockerfile
FROM node:20-alpine
RUN addgroup -g 1000 appgroup && adduser -u 1000 -G appgroup -D appuser
USER appuser
CMD ["node", "server.js"]   # ✅ processus exécuté en utilisateur non-root
```

> [!warning] Root par défaut = surface d'attaque élargie
> Sans instruction `USER` explicite, un conteneur compromis donne immédiatement des privilèges root à l'intérieur de celui-ci. Voir [[Docker 08 — Sécurité des conteneurs]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Données perdues à la suppression du conteneur | Déclarer un volume nommé explicite |
| Cache de build cassé à chaque commit | Copier les dépendances avant le code |
| Port "exposé" mais inaccessible | Utiliser `-p` ou `ports:`, pas seulement `EXPOSE` |
| Conteneurs qui ne se trouvent pas par nom | Réseau bridge personnalisé ou Compose |
| Volumes supprimés par accident | Éviter `down -v` sauf intention explicite |
| Déploiement imprévisible avec :latest | Toujours une version explicite en production |
| Confusion entre créer et entrer dans un conteneur | `run` crée, `exec` entre dans l'existant |
| Conteneur exécuté en root sans le savoir | Toujours déclarer un `USER` non-root explicite |
