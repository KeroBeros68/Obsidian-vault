#devops #docker #réseaux

## Les drivers réseau

Docker attache chaque conteneur à un réseau virtuel via un **driver**, choisi selon le besoin d'isolation, de performance, ou de portée (un seul hôte ou plusieurs).

| Driver | Portée | Cas d'usage |
|--------|--------|-------------|
| **bridge** (défaut) | Un seul hôte | Communication entre conteneurs sur la même machine |
| **host** | Un seul hôte | Performance maximale, pas d'isolation réseau |
| **none** | Aucune | Isolation totale (tâches sans accès réseau) |
| **overlay** | Plusieurs hôtes | Swarm, conteneurs répartis sur plusieurs machines |

## Bridge par défaut vs bridge personnalisé

```bash
# Bridge par défaut — communication uniquement par IP
docker run -d --name web nginx
docker run -d --name app myapp
# 'app' ne peut PAS résoudre 'web' par son nom

# Bridge personnalisé — résolution DNS automatique par nom
docker network create my-net
docker run -d --network my-net --name web nginx
docker run -d --network my-net --name app myapp
# 'app' peut faire ping vers 'web' directement
```

Le bridge par défaut isole les conteneurs des autres réseaux et de l'extérieur, mais ne fournit **pas** de résolution DNS par nom — seul un réseau bridge créé explicitement (`docker network create`) l'offre.

## EXPOSE vs publication de port

```dockerfile
EXPOSE 3000   # documentation uniquement — ne publie rien
```

```bash
# Seule cette commande publie réellement le port vers l'hôte
docker run -p 8080:3000 myapp
```

`EXPOSE` dans un Dockerfile sert uniquement de documentation pour les autres développeurs et certains outils — il ne rend le port accessible nulle part. Seul `-p` (ou `ports:` en Compose) ouvre réellement un accès depuis l'hôte.

## Cas particuliers

> [!warning] host = perte d'isolation
> Le mode `--network host` fait partager au conteneur la pile réseau complète de l'hôte (mêmes ports, mêmes interfaces). Gain de performance réel, mais surface d'attaque élargie — à réserver à des cas précis (ex. outils de monitoring du host).

> [!tip] Toujours un bridge personnalisé
> Pour des conteneurs qui doivent se parler par nom, créer systématiquement un réseau bridge personnalisé plutôt que de compter sur le bridge par défaut.

> [!info] DNS interne
> Sur un réseau bridge personnalisé, Docker fournit un serveur DNS interne (`127.0.0.11`) qui résout automatiquement les noms de conteneurs.
