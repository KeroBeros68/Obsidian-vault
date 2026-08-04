#devops #pure-ftpd #avancé

## Des comptes FTP sans compte système

Un utilisateur virtuel existe uniquement dans la base Pure-FTPd — il n'a ni entrée dans `/etc/passwd`, ni shell, ni possibilité de connexion SSH. C'est le mécanisme recommandé pour tout accès FTP dont le seul besoin est le transfert de fichiers, sans exposer un compte système complet.

## pure-pw : gérer les comptes

```bash
sudo pure-pw useradd alice -u ftpuser -g ftpgroup -d /home/ftp/alice
```

```
Password: ********
Enter it again: ********
```

| Option | Rôle |
|--------|------|
| `-u` | Utilisateur système sous lequel les fichiers d'`alice` seront réellement possédés |
| `-g` | Groupe système associé |
| `-d` | Répertoire racine assigné à ce compte (voir [[Pure-FTPd 04 — Chroot & confinement des utilisateurs]]) |

```bash
sudo pure-pw list                 # Liste tous les comptes virtuels
sudo pure-pw show alice           # Détails d'un compte
sudo pure-pw passwd alice         # Change le mot de passe
sudo pure-pw userdel alice        # Supprime le compte
```

## Deux fichiers, deux rôles

```bash
/etc/pure-ftpd/pureftpd.passwd   # Texte brut, source de vérité, modifié par pure-pw useradd/userdel
/etc/pure-ftpd/pureftpd.pdb      # Binaire indexé, effectivement lu par le démon
```

> [!warning] Toute modification exige une reconstruction de la base binaire
> `pure-pw useradd`/`userdel`/`passwd` modifient le fichier texte `pureftpd.passwd`, mais le démon Pure-FTPd ne lit que la base binaire indexée `pureftpd.pdb`. Sans régénération, les changements restent invisibles pour les connexions en cours et futures :

```bash
sudo pure-pw mkdb
```

> [!tip] Automatiser la régénération
> Certaines installations placent `pure-pw mkdb` dans un timer ou l'exécutent systématiquement après tout script d'administration des comptes, pour éviter l'oubli — une base non régénérée après un `userdel` est un classique piège de sécurité (un compte supprimé côté texte reste actif côté démon).

## Activer PureDB comme backend d'authentification

```bash
echo "yes" | sudo tee /etc/pure-ftpd/conf/PureDB
echo "/etc/pure-ftpd/pureftpd.pdb" | sudo tee -a /etc/pure-ftpd/conf/PureDB
sudo systemctl restart pure-ftpd
```

Voir [[Pure-FTPd 01 — Le mécanisme de configuration (wrapper Debian)]] pour la mécanique complète des fichiers de configuration, et [[Pure-FTPd 03 — Backends d'authentification & priorité]] pour faire cohabiter PureDB avec d'autres sources de comptes (Unix, MySQL, LDAP).

## Pour aller plus loin

Une fois les comptes virtuels créés, leur confinement à un répertoire précis (et les subtilités du chroot) sont couverts dans [[Pure-FTPd 04 — Chroot & confinement des utilisateurs]].

Sources : [pure-pw: Manage virtual users files for Pure-FTPd — systutorials.com](https://www.systutorials.com/docs/linux/man/8-pure-pw/), [README.Virtual-Users — jedisct1/pure-ftpd (GitHub)](https://github.com/jedisct1/pure-ftpd/blob/master/README.Virtual-Users)
