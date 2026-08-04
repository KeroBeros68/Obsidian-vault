#devops #pure-ftpd #sécurité #avancé

## Synthèse : les couches déjà couvertes

Le durcissement d'une instance Pure-FTPd combine plusieurs mécanismes indépendants, chacun détaillé dans une fiche dédiée de ce module : confinement (chroot), chiffrement du transport (TLS), contrôle des backends d'authentification, limitation des ressources par compte.

## Désactiver les comptes anonymes et les comptes système inutiles

```bash
echo "yes" | sudo tee /etc/pure-ftpd/conf/NoAnonymous
sudo systemctl restart pure-ftpd
```

Voir [[FTP 03 — Authentification & FTP anonyme]] pour les risques génériques d'un accès anonyme mal restreint, transposables directement à Pure-FTPd.

## Bannir les connexions root et limiter par IP

```bash
echo "10" | sudo tee /etc/pure-ftpd/conf/MaxClientsPerIP
```

> [!warning] Ne jamais exposer un compte système à privilèges via FTP
> Un compte `root` (ou tout compte à privilèges élevés) ne devrait jamais être atteignable via un backend d'authentification Unix actif sur le serveur FTP — voir [[Pure-FTPd 03 — Backends d'authentification & priorité]] pour désactiver ce backend entièrement si seuls des comptes virtuels PureDB doivent se connecter.

## Masquer la bannière de version

```bash
# Personnaliser le message de bienvenue plutôt que d'exposer la version exacte
echo "/etc/pure-ftpd/welcome.txt" | sudo tee /etc/pure-ftpd/conf/FortunesFile
```

> [!tip] Une version exacte affichée facilite le ciblage d'une vulnérabilité connue
> Comme pour tout service exposé, révéler la version précise du logiciel dans la bannière de connexion aide un attaquant à cibler une CVE connue pour cette version précise — masquer ou personnaliser ce message reste une mesure de défense en profondeur simple à appliquer.

## Checklist de durcissement

| Vérification | Commande | Attendu |
|----------------|----------|---------|
| Chroot actif pour tous | `cat /etc/pure-ftpd/conf/ChrootEveryone` | `yes` |
| TLS forcé | `cat /etc/pure-ftpd/conf/TLS` | `2` |
| Backend Unix désactivé si non voulu | `ls /etc/pure-ftpd/auth/` | Seul PureDB (ou le backend voulu) présent |
| Accès anonyme désactivé (sauf besoin explicite) | `cat /etc/pure-ftpd/conf/NoAnonymous` | `yes` |
| Plage de ports passifs restreinte | `cat /etc/pure-ftpd/conf/PassivePortRange` | Plage définie, pas la plage éphémère par défaut |
| Limite de connexions par IP | `cat /etc/pure-ftpd/conf/MaxClientsPerIP` | Valeur raisonnable définie |
| Base PureDB à jour après toute modification | `pure-pw list` vs contenu réel attendu | Cohérent, `mkdb` exécuté récemment |

## Pour aller plus loin

Cela conclut le module Pure-FTPd — voir [[Pure-FTPd — Index des fiches]] pour une vue d'ensemble, ou [[FTP — Index des fiches]] pour revoir les concepts protocolaires génériques sur lesquels ce module s'appuie.

Sources : [Installation and Configuration of Pure-FTPd — Edoceo](https://edoceo.com/sys/pure-ftpd), [pure-ftpd-wrapper(8) — Debian Manpages](https://manpages.debian.org/testing/pure-ftpd-common/pure-ftpd-wrapper.8.en.html)
