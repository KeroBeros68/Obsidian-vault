#devops #apache #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Virtual Host (VHost)** | Bloc `<VirtualHost>` associant un domaine (`ServerName`) à un dossier de fichiers (`DocumentRoot`) — un site. |
| **MPM (Multi-Processing Module)** | Mécanisme interchangeable de gestion des requêtes en parallèle : `prefork` (un processus par requête), `worker` (processus + threads), `event` (worker + gestion optimisée du keepalive). |
| **`mod_php`** | Module chargeant l'interpréteur PHP directement dans les processus Apache ; non thread-safe, impose le MPM `prefork`. |
| **`a2enmod` / `a2dismod`** | Scripts Debian/Ubuntu activant/désactivant un module via lien symbolique entre `mods-available/` et `mods-enabled/`. N'existent pas sur RHEL/Rocky. |
| **`a2ensite` / `a2dissite`** | Équivalents pour activer/désactiver un site (`sites-available/` → `sites-enabled/`), Debian/Ubuntu uniquement. |
| **`AllowOverride`** | Directive autorisant (`All`) ou interdisant (`None`) qu'un `.htaccess` local surcharge la configuration d'un `<Directory>`. |
| **`.htaccess`** | Fichier de configuration local à un dossier, pris en compte seulement si `AllowOverride` l'autorise. |
| **`ServerAlias`** | Nom de domaine supplémentaire reconnu pour un Virtual Host, en plus de son `ServerName` principal. |
| **`ProxyPass` / `ProxyPassReverse`** | Directives de reverse proxy : transmission de la requête au backend, et réécriture des en-têtes de redirection renvoyés. |
| **`ProxyPreserveHost`** | Transmet le `Host:` original du client au backend plutôt que celui du backend lui-même. |
| **`mod_proxy_balancer`** | Module de répartition de charge entre plusieurs `BalancerMember`, avec choix d'algorithme (`byrequests`, `bytraffic`, `bybusyness`). |
| **`mod_status` / `server-status`** | Module exposant une page de diagnostic sur l'activité courante du serveur (connexions, requêtes en cours). |
| **`mod_deflate`** | Module de compression des réponses (équivalent de `gzip` chez Nginx). |
| **`mod_expires`** | Module ajoutant des en-têtes de cache navigateur par type de fichier. |
| **`ServerTokens` / `ServerSignature`** | Directives masquant la version d'Apache dans l'en-tête `Server` et les pages d'erreur générées. |
| **`apache2ctl` / `httpd`** | Commande de contrôle du service — `apache2ctl` sur Debian/Ubuntu, `httpd` directement sur RHEL/Rocky (`-t` teste, `-M` liste les modules, `-S` liste les Virtual Hosts). |
