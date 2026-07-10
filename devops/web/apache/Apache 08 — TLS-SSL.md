#devops #apache #tls #avancé

## Activer mod_ssl et configurer le VirtualHost HTTPS

```bash
sudo a2enmod ssl   # Debian/Ubuntu — voir Apache 03 pour l'activation de module
sudo systemctl restart apache2
```

```apache
<VirtualHost *:443>
    ServerName exemple.com
    DocumentRoot /var/www/exemple

    SSLEngine On
    SSLCertificateFile /etc/letsencrypt/live/exemple.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/exemple.com/privkey.pem

    # TLS moderne uniquement — ne pas se fier aux valeurs par défaut
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256
    SSLHonorCipherOrder off

    Header always set Strict-Transport-Security "max-age=31536000"

    <Directory /var/www/exemple>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>

# Redirection HTTP → HTTPS
<VirtualHost *:80>
    ServerName exemple.com
    Redirect permanent / https://exemple.com/
</VirtualHost>
```

| Directive | Rôle |
|-----------|------|
| `SSLProtocol` | Liste explicite des protocoles autorisés — équivalent de `ssl_protocols` chez Nginx (voir [[Nginx 08 — TLS-SSL]]) |
| `SSLCipherSuite` | Suites de chiffrement autorisées pour TLS 1.2 (TLS 1.3 impose ses propres suites, non configurables ici) |
| `SSLHonorCipherOrder off` | Laisse le client choisir l'ordre de préférence des suites — recommandation actuelle avec des suites modernes, plutôt que forcer l'ordre du serveur |
| `Header always set Strict-Transport-Security` | HSTS — force le navigateur à n'utiliser que HTTPS pour ce domaine (voir la mise en garde dans [[Apache 10 — Certificats Let's Encrypt avec Certbot]]) |

> [!warning] Jamais SSLv3, TLSv1 ou TLSv1.1 en production
> Ces versions sont vulnérables à des attaques connues et ne sont plus recommandées par aucune référence de sécurité actuelle — comportement identique au même piège côté Nginx (voir [[Nginx — Pièges classiques]]).

## Générer une configuration TLS de référence

Le [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) produit une configuration `SSLProtocol`/`SSLCipherSuite` adaptée à la version exacte d'Apache et d'OpenSSL installée — préférable à copier une configuration trouvée en ligne sans vérifier sa date.

## Cas particuliers

> [!info] mod_ssl vs le certificat lui-même
> `mod_ssl` fournit le mécanisme de chiffrement ; le certificat et sa clé (`SSLCertificateFile`/`SSLCertificateKeyFile`) restent à obtenir séparément — voir [[Apache 10 — Certificats Let's Encrypt avec Certbot]] pour l'obtention pratique via Let's Encrypt.

> [!warning] Un VirtualHost :443 sans redirection laisse HTTP actif
> Sans le second `VirtualHost *:80` avec `Redirect permanent`, le site reste accessible en HTTP non chiffré en parallèle du HTTPS — les deux doivent être définis explicitement, Apache ne redirige rien par défaut.
