#devops #pure-ftpd #avancé

## Quotas par utilisateur : limiter l'espace disque et le nombre de fichiers

```bash
sudo pure-pw usermod alice -n 1000 -N 500M
sudo pure-pw mkdb
```

| Option `pure-pw` | Rôle |
|--------------------|------|
| `-n` | Nombre maximum de fichiers |
| `-N` | Taille maximale cumulée des fichiers de l'utilisateur |

> [!warning] Le quota s'applique après STOR, pas avant
> Pure-FTPd vérifie le quota une fois le transfert reçu, pas en amont — un fichier plus volumineux que l'espace restant peut être intégralement transféré avant d'être rejeté. Sur des connexions lentes ou des fichiers très volumineux, cela consomme de la bande passante pour un transfert qui échouera de toute façon. Combiner avec `MaxDiskUsage` (activation globale du contrôle de quota) pour un comportement cohérent.

```bash
echo "yes" | sudo tee /etc/pure-ftpd/conf/Quota
```

## Ratio upload/download : forcer un équilibre

```bash
sudo pure-pw usermod alice -q 1:5
```

Un ratio `1:5` signifie qu'1 Mo téléversé autorise 5 Mo de téléchargement — un mécanisme hérité de l'époque des sites d'échange communautaires (*warez*), toujours utile pour éviter qu'un compte ne serve exclusivement à distribuer du contenu sans jamais contribuer.

## Limitation de bande passante (throttling)

```bash
sudo pure-pw usermod alice -t 50 -T 200
```

| Option | Rôle |
|--------|------|
| `-t` | Débit maximum en upload, en Ko/s |
| `-T` | Débit maximum en download, en Ko/s |

> [!tip] Un throttling global plutôt que par utilisateur
> Pour une limite appliquée à tous les comptes plutôt qu'individuellement, les fichiers d'option globaux (`/etc/pure-ftpd/conf/`) offrent un équivalent à l'échelle du serveur, utile pour protéger la bande passante totale disponible plutôt que de dépendre d'une configuration cohérente répétée sur chaque compte.

## Limiter le nombre de connexions simultanées

```bash
echo "10" | sudo tee /etc/pure-ftpd/conf/MaxClientsPerIP
echo "50" | sudo tee /etc/pure-ftpd/conf/MaxClientsNumber
```

`MaxClientsPerIP` protège contre un client (ou un script) qui ouvrirait un grand nombre de connexions simultanées depuis la même adresse ; `MaxClientsNumber` plafonne la charge totale du serveur, tous clients confondus.

## Après toute modification via pure-pw

```bash
sudo pure-pw mkdb   # Indispensable — voir [[Pure-FTPd 02 — Utilisateurs virtuels avec PureDB]]
```

## Pour aller plus loin

Le mode passif et les subtilités de pare-feu, déjà couvertes de façon générique pour FTP, s'appliquent à Pure-FTPd avec ses propres options — détaillées dans [[Pure-FTPd 07 — Mode passif & pare-feu avec Pure-FTPd]].

Sources : [pure-pw: Manage virtual users files for Pure-FTPd — systutorials.com](https://www.systutorials.com/docs/linux/man/8-pure-pw/), [Installation and Configuration of Pure-FTPd — Edoceo](https://edoceo.com/sys/pure-ftpd)
