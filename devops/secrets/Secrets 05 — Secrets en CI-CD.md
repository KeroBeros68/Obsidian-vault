#devops #secrets #ci-cd

## Stockage des secrets côté plateforme CI

Les plateformes CI proposent un espace de stockage de secrets séparé du dépôt de code, injecté dans le pipeline sous forme de variables d'environnement au moment de l'exécution.

| Plateforme | Mécanisme | Portée possible |
|-------------|-----------|-------------------|
| GitHub Actions | Repository / Organization / Environment secrets | Dépôt entier, organisation, ou environnement de déploiement précis |
| GitLab CI | CI/CD Variables (masquées, "protected") | Projet, groupe, ou instance |

```yaml
# GitHub Actions
jobs:
  deploy:
    steps:
      - run: ./deploy.sh
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
```

```yaml
# GitLab CI — variable définie dans Settings > CI/CD > Variables, marquée "Masked" et "Protected"
deploy:
  script:
    - ./deploy.sh
  # $API_TOKEN injectée automatiquement dans l'environnement du job
```

## Relier un secret CI à un secret de build Docker

Un secret stocké côté plateforme CI doit encore être transmis à `docker build --secret` (voir [[Secrets 04 — .dockerignore & hygiène de build]]) pour profiter du montage temporaire de BuildKit plutôt que d'atterrir dans une couche de l'image :

```yaml
# GitLab CI
build:
  stage: build
  script:
    - echo "$NPM_TOKEN" > .npmrc          # fichier temporaire, jamais commité
    - docker build --secret id=npmrc,src=.npmrc -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - rm -f .npmrc                         # nettoyage explicite du fichier temporaire
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

```yaml
# GitHub Actions — docker/build-push-action gère nativement le passage de secrets
- uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: ghcr.io/${{ github.repository }}:latest
    secrets: |
      "npm_token=${{ secrets.NPM_TOKEN }}"
```

> [!info] GitHub Actions évite le fichier temporaire
> `docker/build-push-action` transmet directement `secrets.NPM_TOKEN` à BuildKit sans passer par un fichier intermédiaire sur le disque du runner — contrairement au pattern GitLab CI ci-dessus, qui doit créer puis supprimer `.npmrc` explicitement.

## Les limites du masquage automatique

Les deux plateformes masquent (remplacent par `***`) toute occurrence **littérale** d'un secret dans les logs — mais ce masquage est un filtre texte, pas une garantie absolue.

> [!warning] Le masquage contourné par transformation
> Un secret encodé (`base64`), inversé, ou découpé caractère par caractère avant d'être affiché échappe au masquage littéral, puisque la chaîne qui apparaît dans les logs ne correspond plus exactement à la valeur secrète stockée. Éviter tout traitement d'un secret qui pourrait finir dans une sortie de log (`echo`, `print`, sortie d'erreur d'un outil).

## Secrets statiques vs identité fédérée (OIDC)

| | Secret statique (clé d'accès long-lived) | OIDC / identité fédérée |
|---|---------------------------------------------|---------------------------|
| Durée de vie | Longue, jusqu'à révocation manuelle | Jeton de quelques minutes, généré à chaque run |
| Stockage nécessaire | Oui, dans le magasin de secrets CI | Non — la confiance est établie directement entre la plateforme CI et le fournisseur cloud |
| Risque en cas de fuite du log | Élevé, réutilisable jusqu'à révocation | Faible, le jeton expire en quelques minutes |

```yaml
# GitHub Actions → AWS via OIDC, sans clé d'accès stockée
permissions:
  id-token: write
steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789012:role/deploy-role
      aws-region: eu-west-3
```

## Cas particuliers

> [!info] Les secrets ne sont pas exposés aux PR de forks
> Par défaut, GitHub Actions et GitLab CI n'injectent pas les secrets protégés dans les pipelines déclenchés par une pull/merge request venant d'un fork externe — mesure de sécurité contre l'exfiltration via un pipeline modifié par un contributeur non fiable.

> [!tip] Préférer OIDC dès que le fournisseur cloud le supporte
> AWS, GCP et Azure supportent tous l'authentification fédérée depuis GitHub Actions et GitLab CI. Le gain (aucun secret statique à stocker ni à faire tourner) dépasse largement le coût de configuration initial pour tout déploiement récurrent.

> [!warning] Un secret CI reste un secret partagé
> Un secret au niveau "organisation" est accessible à tous les dépôts de cette organisation. Restreindre la portée (environnement, projet) au strict nécessaire limite le rayon d'impact d'un dépôt compromis.
