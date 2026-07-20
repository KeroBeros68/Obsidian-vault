#devops #web #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Serveur web** | Logiciel qui reçoit des requêtes HTTP, va chercher le contenu demandé (fichier ou application) et renvoie une réponse. |
| **Requête / réponse HTTP** | Le dialogue de base du web : le client envoie une requête (méthode + chemin + en-têtes), le serveur renvoie une réponse (code de statut + en-têtes + contenu). |
| **Code de statut HTTP** | Nombre à 3 chiffres indiquant le résultat d'une requête (`200` OK, `301`/`302` redirection, `403` interdit, `404` introuvable, `500` erreur serveur, `502` mauvaise passerelle, `504` délai dépassé côté backend). |
| **Virtual host** | Mécanisme permettant à plusieurs domaines de partager la même adresse IP ; le serveur route selon l'en-tête `Host:` de la requête. |
| **Reverse proxy** | Le serveur web reçoit une requête publique et la transmet à une application backend qui tourne sur un port interne, transparent pour le client. |
| **HTTPS / TLS** | Chiffrement des communications entre client et serveur, empêchant l'interception des données en clair par un intermédiaire réseau. |
| **HSTS (Strict-Transport-Security)** | En-tête HTTP forçant le navigateur à n'utiliser que HTTPS pour un domaine donné, une fois activé. |
| **Cache (HTTP)** | Stockage d'une réponse déjà calculée pour éviter de la recalculer à chaque requête identique. |
| **Compression (gzip)** | Réduction de la taille des données transmises sur le réseau, au prix d'un léger coût de calcul. |
| **`access.log`** | Journal listant toutes les requêtes reçues par le serveur — utile pour analyser le trafic. |
| **`error.log`** | Journal des erreurs du serveur — premier réflexe de diagnostic face à un comportement inattendu. |
| **`.htaccess`** | Fichier de configuration locale à un dossier, spécifique à Apache — sans effet sur Nginx ou Caddy. |
| **MPM (Multi-Processing Module)** | Mécanisme de gestion des requêtes en parallèle propre à Apache (processus/threads) — Nginx et Caddy utilisent une architecture événementielle fixe. |
| **Let's Encrypt** | Autorité de certification gratuite délivrant des certificats TLS valides 90 jours, automatisable via Certbot (Nginx/Apache) ou nativement (Caddy). |
| **Certbot** | Outil qui automatise l'obtention et le renouvellement de certificats Let's Encrypt pour Nginx et Apache. |
| **Reload vs restart** | `reload` applique une nouvelle configuration sans couper les connexions en cours ; `restart` arrête puis relance le service, avec une coupure de service. |
