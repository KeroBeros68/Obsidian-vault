#devops #apache #performance #avancé

## Compression avec mod_deflate

Réduit la taille des réponses transmises de 60 à 80% pour du contenu textuel — équivalent de `gzip` chez Nginx (voir [[Nginx 13 — Cache, compression & load balancing]]).

```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/xml
    AddOutputFilterByType DEFLATE text/css application/javascript
    AddOutputFilterByType DEFLATE application/json application/xml
</IfModule>
```

```bash
# Debian/Ubuntu
sudo a2enmod deflate
sudo systemctl restart apache2
```

```bash
# RHEL/Rocky : généralement déjà activé
httpd -M | grep deflate
```

## Headers de cache avec mod_expires

Indique au navigateur combien de temps garder un fichier statique en cache local, sans revalidation auprès du serveur.

```apache
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
</IfModule>
```

> [!tip] Durée de cache selon la fréquence de changement du contenu
> Des images qui changent rarement supportent une durée longue (1 an) ; CSS/JS, plus susceptibles d'évoluer à chaque déploiement, gagnent à une durée plus courte (1 mois) — sauf si le nom de fichier inclut déjà un hash de version, auquel cas une durée longue redevient sûre.

## Cas particuliers

> [!warning] mod_expires sans versioning des fichiers peut geler une ancienne version
> Un fichier CSS mis en cache pour 1 mois reste servi tel quel par le navigateur du visiteur pendant cette durée, même après une mise à jour côté serveur — un système de versioning dans le nom de fichier (`style.a3f9c2.css`) évite ce problème en changeant l'URL à chaque déploiement plutôt qu'en réduisant la durée de cache.

> [!info] mod_deflate et mod_expires sont indépendants
> Rien n'empêche de combiner les deux : un fichier CSS compressé par `mod_deflate` peut aussi porter un en-tête de cache long via `mod_expires` — les deux mécanismes agissent sur des aspects différents de la réponse HTTP.
