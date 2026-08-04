#devops #pure-ftpd #fondamentaux

## Pure-FTPd natif : des options en ligne de commande, pas un fichier

En amont, Pure-FTPd se lance avec des options passées directement au démon :

```bash
pure-ftpd --daemonize --chrooteveryone --tls=1 --passiveportrange 49152:49252
```

Aucun fichier `pure-ftpd.conf` n'est lu nativement — c'est un choix de conception assumé du projet, à l'opposé de la plupart des autres serveurs FTP ou web.

## Le wrapper Debian/Ubuntu : deux couches par-dessus ce modèle

Pour rendre ce fonctionnement gérable en production, le paquet `pure-ftpd-common` ajoute un utilitaire, `pure-ftpd-wrapper`, qui traduit des fichiers de configuration en ces mêmes arguments avant de démarrer le démon.

### Un fichier par option, dans /etc/pure-ftpd/conf/

```bash
/etc/pure-ftpd/conf/
├── ChrootEveryone      # contient : yes
├── PassivePortRange    # contient : 49152 49252
├── MaxClientsPerIP     # contient : 10
└── TLS                 # contient : 1
```

```bash
echo "yes" | sudo tee /etc/pure-ftpd/conf/ChrootEveryone
echo "49152 49252" | sudo tee /etc/pure-ftpd/conf/PassivePortRange
sudo systemctl restart pure-ftpd
```

> [!info] Le nom du fichier est le nom de l'option
> Chaque nom de fichier dans `/etc/pure-ftpd/conf/` correspond exactement à une option en ligne de commande de Pure-FTPd (sans les tirets, en respectant la casse attendue) — une seule ligne de valeur par fichier, les lignes vides et les commentaires (`#`) étant ignorés. `pure-ftpd-wrapper --show-options` liste l'ensemble des options reconnues.

### Les valeurs booléennes acceptées

```bash
echo "yes" > /etc/pure-ftpd/conf/ChrootEveryone   # ou : 1, On
echo "no"  > /etc/pure-ftpd/conf/ChrootEveryone   # ou : 0, Off
```

> [!warning] Supprimer le fichier désactive l'option, un fichier vide ne suffit pas toujours
> Pour désactiver une option booléenne, écrire explicitement `no`/`0`/`off` dans le fichier — ou le supprimer entièrement. Un fichier présent mais vide peut être interprété différemment selon le type d'option ; ne pas s'y fier par défaut.

### auth/ : la priorité des backends d'authentification

```bash
/etc/pure-ftpd/auth/
└── 30puredb -> /usr/lib/pure-ftpd/pureauthPuredb
```

Le répertoire `/etc/pure-ftpd/auth/` ne contient que des **liens symboliques**, examinés dans l'ordre alphabétique de leur nom — le préfixe numérique (`30`, `40`...) sert précisément à contrôler cet ordre. Le premier backend qui reconnaît l'utilisateur l'authentifie ; les autres ne sont pas consultés.

> [!tip] Un ordre explicite plutôt qu'une liste dans un fichier
> Pour prioriser l'authentification via PureDB (voir [[Pure-FTPd 02 — Utilisateurs virtuels avec PureDB]]) avant les comptes système Unix, il suffit de nommer son lien avec un préfixe numérique inférieur à celui du backend Unix — l'ordre alphabétique des noms de fichiers *est* la politique de priorité.

## /etc/pure-ftpd/pure-ftpd.conf : une alternative plus classique

Le paquet propose aussi un fichier unique, de forme plus traditionnelle (`option valeur` par ligne), lu par `pure-ftpd-wrapper` en complément ou à la place des fichiers individuels selon la configuration du système.

```conf
# /etc/pure-ftpd/pure-ftpd.conf
ChrootEveryone yes
MaxClientsPerIP 10
```

> [!warning] Les deux mécanismes coexistent, ne pas les mélanger sans comprendre l'ordre de priorité
> Utiliser exclusivement l'un ou l'autre (soit `pure-ftpd.conf`, soit `/etc/pure-ftpd/conf/`) pour une option donnée évite toute ambiguïté sur la valeur réellement appliquée après un redémarrage. `pure-ftpd-wrapper --show-options` et l'inspection de `ps aux | grep pure-ftpd` (pour voir les arguments réellement passés au démon) restent les meilleurs outils de vérification en cas de doute.

## Vérifier ce qui est réellement appliqué

```bash
pure-ftpd-wrapper --show-options | less
ps aux | grep pure-ftpd
```

## Pour aller plus loin

La création de comptes FTP indépendants des comptes système, via PureDB, est couverte dans [[Pure-FTPd 02 — Utilisateurs virtuels avec PureDB]].

Sources : [pure-ftpd-wrapper(8) — Debian Manpages](https://manpages.debian.org/testing/pure-ftpd-common/pure-ftpd-wrapper.8.en.html), [Installation and Configuration of Pure-FTPd — Edoceo](https://edoceo.com/sys/pure-ftpd)
