#devops #pure-ftpd #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **`pure-ftpd-wrapper`** | Utilitaire (paquet `pure-ftpd-common`) traduisant les fichiers de configuration Debian/Ubuntu en arguments de ligne de commande pour le démon Pure-FTPd natif. |
| **`/etc/pure-ftpd/conf/`** | Répertoire où chaque fichier correspond à une option du démon — un fichier, une valeur, sans fichier monolithique classique. |
| **`/etc/pure-ftpd/auth/`** | Répertoire de liens symboliques déterminant, par ordre alphabétique, la priorité des backends d'authentification interrogés. |
| **PureDB** | Système de comptes virtuels propre à Pure-FTPd, indépendant des comptes système Unix, géré via `pure-pw`. |
| **`pure-pw`** | Utilitaire en ligne de commande gérant les comptes virtuels : création, modification, suppression, quotas, ratio, throttling. |
| **`pureftpd.passwd` / `pureftpd.pdb`** | Respectivement le fichier texte source de vérité et la base binaire indexée effectivement lue par le démon — `pure-pw mkdb` synchronise la seconde depuis le premier. |
| **`ChrootEveryone`** | Option confinant chaque utilisateur connecté à son répertoire assigné, traité comme racine `/` de sa session. |
| **`TLS` (0/1/2)** | Option à trois niveaux : désactivé, accepté en option, ou exigé pour toute connexion. |
| **`pure-ftpd.pem`** | Fichier PEM combiné (certificat + clé privée) attendu par Pure-FTPd pour activer TLS, à l'emplacement conventionnel `/etc/ssl/private/`. |
| **Ratio upload/download** | Contrainte (`pure-pw usermod -q`) imposant un volume de téléversement minimum en proportion du téléchargement autorisé. |
| **`PassivePortRange`** | Option restreignant la plage de ports utilisée pour les connexions de données en mode passif, pour une règle de pare-feu praticable. |
| **`ForcePassiveIP`** | Option forçant l'adresse IP annoncée dans les réponses `PASV`, indispensable quand le serveur est lui-même derrière un NAT. |
| **`MaxClientsPerIP` / `MaxClientsNumber`** | Limites de connexions simultanées, respectivement par adresse IP source et pour l'ensemble du serveur. |
| **`FortunesFile`** | Option personnalisant le message de bienvenue affiché à la connexion, utile pour ne pas exposer la version exacte du logiciel. |
