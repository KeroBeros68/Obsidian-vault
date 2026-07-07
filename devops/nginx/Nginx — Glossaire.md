#devops #nginx #glossaire #référence

| Terme | Définition |
|-------|-----------|
| **Master process** | Processus principal de Nginx ; lit la configuration, ouvre les sockets d'écoute, gère le cycle de vie des workers. Ne traite aucune requête lui-même. |
| **Worker process** | Processus qui traite réellement les connexions, via une boucle d'événements asynchrone. Un par cœur CPU en général (`worker_processes auto`). |
| **Contexte** | Bloc `{ }` de la configuration (`http`, `server`, `location`...) ; les directives héritent du contexte parent sauf redéfinition locale. |
| **Directive** | Instruction de configuration (`listen`, `root`, `fastcgi_pass`...), valide uniquement dans certains contextes. |
| **Reverse proxy** | Serveur qui reçoit la requête du client à sa place et la transmet à un backend, transparent pour le client. |
| **Load balancer** | Répartit les requêtes entre plusieurs instances backend selon un algorithme (round-robin, least_conn...). |
| **FastCGI** | Protocole binaire (distinct de HTTP) utilisé par Nginx pour déléguer l'exécution d'un script à un processus applicatif comme PHP-FPM. |
| **PHP-FPM** | Gestionnaire de processus FastCGI pour PHP ; exécute les scripts `.php` reçus via `fastcgi_pass`. |
| **`try_files`** | Directive qui teste une liste de chemins dans l'ordre et sert le premier qui existe, avec un dernier argument utilisé sans condition comme repli. |
| **`location`** | Bloc qui définit comment traiter les requêtes dont l'URI correspond à un motif donné (exact, préfixe, regex). |
| **Virtual hosting** | Faire cohabiter plusieurs sites (`server_name` différents) sur la même instance Nginx et souvent le même port. |
| **SNI (Server Name Indication)** | Extension TLS qui transmet le nom de domaine demandé dès la poignée de main, permettant à Nginx de choisir le bon certificat/`server` avant même de voir la requête HTTP. |
| **Terminaison TLS** | Le fait que le chiffrement/déchiffrement TLS s'arrête à Nginx, qui transmet ensuite la requête en clair au backend interne. |
| **C10K** | Problème historique de tenir 10 000 connexions simultanées avec des ressources limitées ; à l'origine de la conception événementielle de Nginx. |
| **epoll / kqueue** | Mécanismes du noyau (Linux / BSD) permettant à un worker de surveiller des milliers de connexions sans thread dédié par connexion. |
