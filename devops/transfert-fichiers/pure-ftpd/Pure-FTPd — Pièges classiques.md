#devops #pure-ftpd #pièges #erreurs #debugging

## 🪤 Piège 1 — Modifier un compte pure-pw sans régénérer la base

```bash
sudo pure-pw userdel alice
# ❌ alice peut encore se connecter : pureftpd.pdb n'a pas été régénéré
```

> [!warning] Le démon ne lit que la base binaire, jamais le fichier texte
> `pure-pw useradd`/`userdel`/`passwd` ne modifient que `pureftpd.passwd`. Sans `pure-pw mkdb`, le démon continue de servir l'ancienne base binaire `pureftpd.pdb` — un compte supprimé reste actif jusqu'à la régénération. Voir [[Pure-FTPd 02 — Utilisateurs virtuels avec PureDB]].

---

## 🪤 Piège 2 — Laisser le backend Unix actif à côté de PureDB

```bash
ls /etc/pure-ftpd/auth/
30puredb  60unix   # ❌ 60unix jamais retiré
```

> [!warning] N'importe quel compte système peut alors se connecter en FTP
> Si seuls des comptes virtuels PureDB doivent avoir accès, retirer le lien symbolique du backend Unix dans `/etc/pure-ftpd/auth/` — sinon un compte système jamais destiné au FTP (y compris un compte de service) peut s'y authentifier avec son mot de passe système. Voir [[Pure-FTPd 03 — Backends d'authentification & priorité]].

---

## 🪤 Piège 3 — Un répertoire de chroot possédé par l'utilisateur lui-même

```bash
ls -ld /home/ftp/alice
drwxr-xr-x  2 alice alice  4096   # ❌ le compte confiné possède sa propre racine de chroot
```

> [!warning] Un vecteur classique de sortie de chroot
> Le répertoire racine du chroot doit appartenir à `root` (ou un compte que l'utilisateur ne contrôle pas), seuls les sous-répertoires doivent lui appartenir en écriture. Voir [[Pure-FTPd 04 — Chroot & confinement des utilisateurs]].

---

## 🪤 Piège 4 — Confondre les deux mécanismes de configuration

```bash
# Une même option définie différemment aux deux endroits
cat /etc/pure-ftpd/pure-ftpd.conf | grep ChrootEveryone   # yes
cat /etc/pure-ftpd/conf/ChrootEveryone                     # no
```

> [!warning] Ne pas mélanger les deux mécanismes pour une même option
> `pure-ftpd.conf` (fichier unique) et `/etc/pure-ftpd/conf/` (un fichier par option) coexistent — utiliser l'un ou l'autre par option, jamais les deux, pour éviter toute ambiguïté sur la valeur réellement appliquée. `pure-ftpd-wrapper --show-options` lève le doute. Voir [[Pure-FTPd 01 — Le mécanisme de configuration (wrapper Debian)]].

---

## 🪤 Piège 5 — ForcePassiveIP sans rediriger toute la plage de ports

```bash
# Redirection incomplète (conteneur, box internet)
# Port 21 redirigé, mais pas 49152-49252
```

> [!warning] La connexion réussit, tout transfert échoue
> `ForcePassiveIP` annonce la bonne adresse publique, mais si la plage `PassivePortRange` correspondante n'est pas intégralement redirigée/ouverte, tout `RETR`/`STOR` reste bloqué malgré une authentification et une navigation fonctionnelles — le symptôme caractéristique déjà documenté en [[FTP — Pièges classiques]]. Voir [[Pure-FTPd 07 — Mode passif & pare-feu avec Pure-FTPd]].

---

## Récapitulatif rapide

| Piège | Solution |
|-------|----------|
| Compte modifié via `pure-pw` sans `mkdb` | Toujours régénérer la base binaire après modification |
| Backend Unix actif en plus de PureDB | Retirer le lien symbolique si non désiré |
| Répertoire de chroot possédé par l'utilisateur confiné | La racine du chroot doit appartenir à `root` |
| Option définie différemment dans les deux mécanismes de config | N'utiliser qu'un seul mécanisme par option |
| Plage de ports passifs non entièrement redirigée | Rediriger `PassivePortRange` en entier, pas seulement le port 21 |
