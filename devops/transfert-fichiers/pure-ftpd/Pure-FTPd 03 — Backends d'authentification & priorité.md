#devops #pure-ftpd #avancé

## Plusieurs sources de comptes possibles, une seule à la fois par utilisateur

Pure-FTPd peut authentifier ses utilisateurs contre plusieurs backends différents : PureDB (voir [[Pure-FTPd 02 — Utilisateurs virtuels avec PureDB]]), les comptes système Unix classiques, ou une base externe (MySQL, PostgreSQL, LDAP) pour une gestion centralisée à grande échelle.

| Backend | Cas d'usage typique |
|---------|----------------------|
| PureDB | Comptes FTP-only, simple, sans dépendance externe |
| Unix (`/etc/passwd`) | Réutiliser des comptes système existants tels quels |
| MySQL / PostgreSQL | Gestion centralisée, portail web d'auto-inscription, grand nombre de comptes |
| LDAP | Intégration à un annuaire d'entreprise existant |

## L'ordre de priorité : des liens symboliques, pas une liste

```bash
ls -la /etc/pure-ftpd/auth/
30puredb -> /usr/lib/pure-ftpd/pureauthPuredb
```

Voir [[Pure-FTPd 01 — Le mécanisme de configuration (wrapper Debian)]] pour le détail du mécanisme : `pure-ftpd-wrapper` parcourt `/etc/pure-ftpd/auth/` par ordre alphabétique et interroge chaque backend jusqu'à ce que l'un d'eux reconnaisse l'utilisateur.

```bash
# Faire passer PureDB avant l'authentification Unix
sudo ln -s /usr/lib/pure-ftpd/pureauthPuredb /etc/pure-ftpd/auth/30puredb
sudo ln -s /usr/lib/pure-ftpd/pureauthUnix /etc/pure-ftpd/auth/60unix
```

> [!tip] Le préfixe numérique n'est qu'une convention de tri
> Rien n'oblige à utiliser des dizaines (`30`, `60`...) — seul l'ordre alphabétique des noms de fichiers compte. La convention en dizaines laisse simplement de la place pour insérer un backend supplémentaire plus tard sans renommer les liens existants.

## Un piège classique : deux backends actifs sans le vouloir

> [!warning] Un utilisateur système peut se connecter en FTP sans compte virtuel dédié
> Si le backend Unix reste actif en plus de PureDB, n'importe quel compte système existant (y compris des comptes de service jamais destinés à un accès FTP) peut potentiellement s'authentifier — avec son mot de passe système, et sans passer par le confinement éventuellement prévu pour les comptes virtuels. Désactiver explicitement le backend Unix (retirer le lien correspondant dans `/etc/pure-ftpd/auth/`) si seuls des comptes PureDB doivent pouvoir se connecter.

## Vérifier quel backend a authentifié une connexion

```bash
sudo journalctl -u pure-ftpd -n 100 | grep -i auth
```

Les journaux Pure-FTPd indiquent généralement la source d'authentification utilisée pour chaque connexion réussie — utile pour confirmer qu'un compte attendu en PureDB ne passe pas silencieusement par un autre backend.

## Pour aller plus loin

Quel que soit le backend, le confinement de chaque utilisateur à son propre répertoire reste une couche de sécurité indépendante — détaillée dans [[Pure-FTPd 04 — Chroot & confinement des utilisateurs]].

Sources : [pure-ftpd-wrapper(8) — Debian Manpages](https://manpages.debian.org/testing/pure-ftpd-common/pure-ftpd-wrapper.8.en.html), [README.Virtual-Users — jedisct1/pure-ftpd (GitHub)](https://github.com/jedisct1/pure-ftpd/blob/master/README.Virtual-Users)
