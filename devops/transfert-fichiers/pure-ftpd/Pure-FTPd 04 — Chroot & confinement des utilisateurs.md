#devops #pure-ftpd #sécurité #avancé

## ChrootEveryone : confiner tout le monde par défaut

```bash
echo "yes" | sudo tee /etc/pure-ftpd/conf/ChrootEveryone
sudo systemctl restart pure-ftpd
```

Une fois activé, chaque utilisateur qui se connecte voit son répertoire assigné (`-d` lors de `pure-pw useradd`, voir [[Pure-FTPd 02 — Utilisateurs virtuels avec PureDB]]) traité comme la racine `/` de sa session — il ne peut ni lister, ni naviguer, ni référencer par chemin absolu quoi que ce soit en dehors de ce répertoire.

> [!warning] Sans ChrootEveryone, un compte virtuel peut remonter l'arborescence
> Sans confinement actif, rien n'empêche un client FTP de naviguer avec `CWD ../../..` en dehors du répertoire assigné, jusqu'à atteindre la racine réelle du système de fichiers (dans la limite des permissions Unix du compte système sous-jacent, `-u`/`-g` de `pure-pw useradd`). `ChrootEveryone` est la ligne de défense qui rend ce risque non pertinent.

## Le confinement ne remplace pas les permissions Unix

> [!info] Deux mécanismes indépendants, tous deux nécessaires
> Le chroot limite ce qu'un utilisateur peut **voir** (l'arborescence visible). Les permissions Unix classiques (propriétaire, groupe, droits `rwx`) limitent ce qu'il peut **faire** à l'intérieur de cette arborescence. Un répertoire chrooté mais possédé par le même utilisateur système que le processus Pure-FTPd, avec des droits d'écriture larges, reste vulnérable à une modification indésirable de ses propres fichiers de configuration si ceux-ci se trouvaient (par erreur) dans le même sous-arbre.

## Un piège structurel : le premier niveau chrooté doit appartenir à root

```bash
ls -ld /home/ftp/alice
drwxr-xr-x  2 root root  4096  /home/ftp/alice
```

> [!warning] Un répertoire racine chrooté inscriptible par l'utilisateur lui-même est une faille
> Si l'utilisateur confiné possède également son propre répertoire racine de chroot (au lieu que celui-ci appartienne à `root`), certaines attaques classiques de sortie de chroot deviennent possibles (recréation de structures système factices dans le répertoire racine confiné). Le répertoire racine du chroot doit appartenir à `root` (ou un compte que l'utilisateur ne contrôle pas) ; seuls les sous-répertoires doivent lui appartenir en écriture.

```bash
sudo mkdir -p /home/ftp/alice/upload
sudo chown root:root /home/ftp/alice
sudo chown alice_uid:ftpgroup /home/ftp/alice/upload
```

## Vérifier le confinement effectif

```bash
ftp alice@serveur
ftp> pwd
257 "/" is the current directory
ftp> cd ..
550 Can't change directory
```

Une tentative de remonter au-delà de la racine assignée doit systématiquement échouer une fois `ChrootEveryone` actif et le répertoire correctement possédé par `root`.

## Pour aller plus loin

Le chiffrement du transport, via TLS, complète ce confinement pour protéger aussi les identifiants et le contenu transféré — détaillé dans [[Pure-FTPd 05 — TLS-FTPS avec Pure-FTPd]].

Sources : [Installation and Configuration of Pure-FTPd — Edoceo](https://edoceo.com/sys/pure-ftpd), [README.Virtual-Users — jedisct1/pure-ftpd (GitHub)](https://github.com/jedisct1/pure-ftpd/blob/master/README.Virtual-Users)
