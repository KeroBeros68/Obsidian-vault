#devops #secrets #pièges #erreurs #debugging

## 🪤 Piège 1 — Secret dans une instruction ENV ou ARG du Dockerfile

```dockerfile
# ❌ Persiste dans les métadonnées de l'image, visible via `docker history`
ENV DB_PASSWORD=hunter2
```

```dockerfile
# ✅ Le secret est injecté à l'exécution, jamais écrit dans l'image
# via un fichier monté ou une variable fournie par l'orchestrateur au lancement
```

> [!warning] docker history n'oublie rien
> Voir [[Secrets 04 — .dockerignore & hygiène de build]].

---

## 🪤 Piège 2 — `.env` committé par erreur dans Git

```bash
git add .          # ❌ .env non exclu par .gitignore, committé avec les vrais secrets
```

> [!tip] Mémo
> `.gitignore` **et** `.dockerignore` doivent tous les deux exclure `.env` et les dossiers de secrets — l'un protège le dépôt, l'autre protège le contexte de build.

---

## 🪤 Piège 3 — Croire que supprimer un fichier de Git supprime le secret

```bash
git rm secret.txt && git commit -m "remove secret"   # ❌ reste dans l'historique
```

> [!warning] La seule remédiation fiable : révoquer et régénérer
> Voir [[Secrets 01 — Le problème des secrets en clair]].

---

## 🪤 Piège 4 — Secret Compose "monté" mais fichier source versionné

```yaml
secrets:
  db_password:
    file: ./secrets/db_password.txt   # ❌ si ce fichier est suivi par Git, aucune protection réelle
```

> [!warning] Le montage ne chiffre rien
> Voir [[Secrets 03 — Docker secrets & Compose]].

---

## 🪤 Piège 5 — Compter sur le masquage automatique des logs CI

```bash
echo $API_TOKEN | base64   # ❌ la sortie encodée échappe au masquage littéral
```

> [!tip] Mémo
> Ne jamais transformer un secret avant une sortie qui finit dans un log. Voir [[Secrets 05 — Secrets en CI-CD]].

---

## 🪤 Piège 6 — Un seul secret partagé par tous les environnements

```
❌ Le même mot de passe DB pour dev, staging et prod
✅ Un secret distinct par environnement — la fuite d'un environnement de dev ne compromet pas la prod
```

> [!warning] Blast radius élargi inutilement
> Mutualiser un secret entre environnements de criticité différente transforme la fuite la moins grave (dev) en incident de production.

---

## 🪤 Piège 7 — Oublier `required=true` sur un secret de build indispensable

```dockerfile
# ❌ Sans required=true, le build réussit même si le secret n'est pas fourni —
#    l'échec ne se révèle qu'à l'exécution, bien après le build
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token 2>/dev/null) npm install
```

```dockerfile
# ✅ Le build échoue immédiatement si --secret id=npm_token n'a pas été passé
RUN --mount=type=secret,id=npm_token,required=true \
    NPM_TOKEN=$(cat /run/secrets/npm_token) npm install
```

> [!warning] Un build "réussi" n'implique pas que le secret était présent
> Sans `required=true`, `docker build` sans `--secret id=npm_token,src=...` produit quand même une image — potentiellement avec des dépendances manquantes ou une configuration incomplète, découverte seulement au déploiement. Voir [[Secrets 04 — .dockerignore & hygiène de build]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Secret dans `ENV`/`ARG` du Dockerfile | `RUN --mount=type=secret` ou injection à l'exécution |
| `.env` committé par erreur | `.gitignore` + `.dockerignore` systématiques |
| Suppression Git censée effacer le secret | Révoquer et régénérer, pas nettoyer l'historique |
| Fichier secret Compose versionné dans Git | Exclure le fichier source, ou utiliser un vrai gestionnaire externe |
| Confiance aveugle dans le masquage CI | Ne jamais transformer un secret avant une sortie loggée |
| Un seul secret pour tous les environnements | Un secret distinct par environnement |
| Build qui réussit sans le secret attendu | `required=true` sur le `--mount=type=secret` |
