#bdd #adminer #intermédiaire #plugins

## Le principe : un fichier de personnalisation, pas une modification du cœur

Adminer charge des plugins via un fichier séparé qui inclut le fichier principal et instancie les extensions souhaitées — le fichier `adminer.php` d'origine n'est jamais modifié directement, ce qui simplifie les mises à jour de version.

```php
<?php
// index.php — point d'entrée personnalisé, à côté d'adminer.php
function adminer_object() {
    include './plugins/plugin.php';
    include './plugins/login-password-less.php';

    $plugins = [
        new AdminerLoginPasswordLess(),
    ];

    include './adminer.php';   // fichier officiel, non modifié
}

adminer_object();
```

## Cas d'usage courants

| Besoin | Plugin type |
|--------|--------------|
| Ajouter un driver pour un moteur non natif | Elasticsearch, MongoDB, Redis, ClickHouse |
| Renforcer l'authentification | Support OTP, restriction par IP intégrée à l'application plutôt qu'au serveur web |
| Personnaliser l'interface | Thème (`ADMINER_DESIGN` en Docker), masquage de certaines bases système |
| Journaliser les actions | Plugins de logging des requêtes exécutées via l'interface |

> [!info] Où trouver les plugins
> La liste officielle des plugins (maintenus par le projet et contribués par la communauté) se trouve sur la page Plugins d'`adminer.org` et dans le répertoire `plugins/` du dépôt GitHub — chaque plugin est un simple fichier PHP à inclure.

## Avec Docker

L'image officielle lit les scripts PHP déposés dans `/var/www/html/plugins-enabled/` et les charge automatiquement — pas besoin de fichier `index.php` personnalisé dans ce cas :

```yaml
services:
  adminer:
    image: adminer
    volumes:
      - ./plugins-enabled:/var/www/html/plugins-enabled
```

## Pour aller plus loin

Un plugin de restriction d'accès complète les mesures serveur (IP whitelist nginx/Apache) décrites dans [[Adminer 03 — Sécurisation]], sans les remplacer — les deux couches restent recommandées en parallèle.

Sources : [Adminer — site officiel](https://www.adminer.org/en/), [Adminer sur GitHub](https://github.com/vrana/adminer/), [Adminer — Docker Hub (image officielle)](https://hub.docker.com/_/adminer/)
