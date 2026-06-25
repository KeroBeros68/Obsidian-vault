#devops #docker #pièges #erreurs #debugging

## 🪤 Piège 1 — Croire que supprimer un conteneur supprime les données

```bash
docker run -d --name db postgres:17   # ❌ pas de volume déclaré
docker rm -f db                        # toutes les données sont perdues
```

> [!warning] Couche RW éphémère
> Sans volume explicite, toutes les données écrites dans le conteneur disparaissent à sa suppression. Voir [[Docker 03 — Volumes & persistance]].

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
> Place en haut du Dockerfile ce qui change rarement (dépendances), en bas ce qui change souvent (code). Voir [[Docker 02 — Dockerfile]].

---

## 🪤 Piège 3 — Confondre EXPOSE et publication de port

```dockerfile
EXPOSE 3000   # ❌ ne rend RIEN accessible depuis l'hôte
```

```bash
docker run -p 8080:3000 myapp   # ✅ seule commande qui publie réellement
```

> [!warning] EXPOSE est documentaire
> `EXPOSE` dans un Dockerfile n'a aucun effet réseau réel. Voir [[Docker 04 — Réseaux]].

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

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Données perdues à la suppression du conteneur | Déclarer un volume nommé explicite |
| Cache de build cassé à chaque commit | Copier les dépendances avant le code |
| Port "exposé" mais inaccessible | Utiliser `-p` ou `ports:`, pas seulement `EXPOSE` |
| Conteneurs qui ne se trouvent pas par nom | Réseau bridge personnalisé ou Compose |
| Volumes supprimés par accident | Éviter `down -v` sauf intention explicite |
