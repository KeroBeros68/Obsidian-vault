#bdd #adminer #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Adminer** | Outil d'administration de bases de données distribué en un fichier PHP unique, alternative à phpMyAdmin, multi-moteurs (MySQL, PostgreSQL, SQLite, MS SQL, Oracle...). |
| **Adminer Editor** | Variante d'Adminer centrée sur l'édition de données pour des utilisateurs non techniques, avec moins de fonctionnalités d'administration structurelle. |
| **`latest.php`** | URL officielle (`adminer.org/latest.php`) pointant toujours vers la dernière version stable — à privilégier au téléchargement d'une version figée. |
| **Driver** | Module Adminer traduisant les actions de l'interface en requêtes spécifiques à un moteur donné (MySQL, PostgreSQL...). |
| **Plugin** | Fichier PHP additionnel étendant Adminer (nouveau driver, authentification renforcée, thème, journalisation) sans modifier le fichier principal. |
| **`plugins-enabled/`** | Répertoire lu automatiquement par l'image Docker officielle pour charger des plugins personnalisés. |
| **`ADMINER_DEFAULT_SERVER`** | Variable d'environnement (image Docker) pré-remplissant le champ serveur du formulaire de connexion. |
| **`ADMINER_DESIGN`** | Variable d'environnement (image Docker) sélectionnant un thème d'interface parmi ceux embarqués. |
| **Rate-limiting (Adminer)** | Ralentissement intégré des tentatives de connexion successives, protection basique contre le brute-force — insuffisant seul en production. |
| **SSRF (Server-Side Request Forgery)** | Classe de vulnérabilité ayant touché Adminer (CVE-2021-21311) : le serveur est forcé à effectuer une requête réseau vers une cible choisie par l'attaquant. |
| **`?script=version`** | Point d'accès permettant de vérifier la version installée — aussi le point d'entrée exploité par CVE-2026-25892 (déni de service). |
