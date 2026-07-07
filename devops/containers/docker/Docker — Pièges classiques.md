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

## 🪤 Piège 9 — depends_on sans condition ne garantit pas la disponibilité du service

```yaml
services:
  api:
    build: .
    depends_on:
      - db   # ❌ attend que le conteneur démarre, pas que la DB soit prête à répondre
  db:
    image: postgres:17
```

```yaml
services:
  api:
    build: .
    depends_on:
      db:
        condition: service_healthy   # ✅ attend que le healthcheck de db soit "healthy"
  db:
    image: postgres:17
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      retries: 5
```

> [!warning] "Démarré" ne veut pas dire "prêt"
> Un conteneur peut être démarré (processus lancé) bien avant que le service qu'il héberge accepte des connexions. Voir [[Docker 07 — Docker Compose]].

---

## 🪤 Piège 10 — Écrire dans un chemin VOLUME après l'avoir déclaré

```dockerfile
FROM mysql:8
VOLUME ["/var/lib/mysql"]
RUN echo "init" > /var/lib/mysql/marker.txt   # ❌ écriture ignorée au lancement du conteneur
```

> [!warning] Le contenu du volume remplace celui de l'image à l'exécution
> Dès qu'un chemin est déclaré par `VOLUME`, tout ce qui a été écrit à cet emplacement par une instruction Dockerfile **après** cette déclaration est ignoré au démarrage du conteneur — le chemin est remplacé par le volume (anonyme ou nommé), pas par le contenu de la couche d'image. Voir [[Docker 03 — Dockerfile]].

---

## 🪤 Piège 11 — Choisir une image minimale sans anticiper ses limites

```dockerfile
FROM scratch
COPY --from=builder /app/myapp /myapp
ENTRYPOINT ["/myapp"]
# ❌ Le moindre appel HTTPS échoue : aucun certificat CA n'est présent dans scratch
```

```dockerfile
FROM gcr.io/distroless/base-debian12
# ❌ Aucun shell : "docker exec ... sh" échoue en cas d'incident en production
```

> [!warning] Minimal ne veut pas dire prêt à l'emploi tel quel
> `scratch` n'embarque aucun certificat CA (copier `/etc/ssl/certs/ca-certificates.crt` depuis le stage de build, ou préférer `distroless/static` qui les inclut). Une image distroless standard n'a pas de shell : le tag `:debug` existe pour un diagnostic ponctuel, mais ne doit jamais remplacer l'image de production. Voir [[Docker 03 — Dockerfile]].

---

## 🪤 Piège 12 — Publier un port sans restreindre l'interface

```bash
docker run -p 3306:3306 mysql   # ❌ équivaut à -p 0.0.0.0:3306:3306 : accessible depuis l'extérieur
```

```bash
docker run -p 127.0.0.1:3306:3306 mysql   # ✅ accessible uniquement depuis l'hôte
```

> [!warning] `-p` sans IP écoute sur toutes les interfaces
> Un service qui ne doit être consulté que par un autre conteneur ou par l'hôte lui-même (base de données, outil d'administration) n'a pas besoin d'être exposé sur une interface publique. Voir [[Docker 06 — Réseaux]].

---

## 🪤 Piège 13 — Croire que `docker.sock:ro` protège quelque chose

```yaml
# ❌ :ro ne bloque aucune écriture, seulement le remplacement du fichier socket
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro
```

```bash
# La preuve : un POST d'arrêt fonctionne quand même à travers ce montage :ro
docker run --rm --user 0 -v /var/run/docker.sock:/var/run/docker.sock:ro \
  curlimages/curl -X POST --unix-socket /var/run/docker.sock \
  "http://localhost/v1.44/containers/$CID/stop"
# 204 — le conteneur s'arrête réellement
```

> [!warning] `:ro` porte sur le fichier, pas sur l'API
> Le socket est un canal de communication HTTP avec le démon : un `POST` d'écriture emprunte le même canal qu'un `GET` de lecture, et `:ro` ne les distingue pas. La seule protection réelle est un `docker-socket-proxy` qui filtre les endpoints. Voir [[Docker 15 — Socket-proxy (sécuriser l'accès au socket)]].

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
| Service qui démarre avant d'être vraiment prêt | `depends_on` avec `condition: service_healthy` + un `healthcheck` |
| Écriture après `VOLUME` ignorée à l'exécution | Initialiser les données via un script d'entrée, pas via le Dockerfile |
| Image minimale (scratch/distroless) qui casse HTTPS ou le debug | Copier les certificats CA, utiliser `:debug` uniquement en diagnostic |
| Port publié accessible depuis l'extérieur sans le vouloir | Préfixer `-p` par `127.0.0.1:` pour un service local uniquement |
| `docker.sock:ro` censé bloquer les écritures | Passer par un `docker-socket-proxy` filtrant, `:ro` seul ne protège rien |
