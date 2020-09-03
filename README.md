# wordpress-http2
The **WordPress 5.5** docker image with apache2.4 (http2). The image based on Ubuntu 20.04, php-fpm-7.4. \
Main target its create image with the possibility to use Apache2 on http2 protocol. The big part of the code was taken from official docker file wordpress:5.5 and php7.3:apache/fpm.

## ENV
The image has same environment variables as official wordpress docker image [Wordpress Docker Hub](https://hub.docker.com/_/wordpress) \
The following environment variables are also honored for configuring your WordPress instance:

- -e WORDPRESS_DB_HOST=...
- -e WORDPRESS_DB_USER=...
- -e WORDPRESS_DB_PASSWORD=...
- -e WORDPRESS_DB_NAME=...
- -e WORDPRESS_TABLE_PREFIX=...
- -e WORDPRESS_AUTH_KEY=..., -e WORDPRESS_SECURE_AUTH_KEY=..., -e WORDPRESS_LOGGED_IN_KEY=..., -e WORDPRESS_NONCE_KEY=..., -e WORDPRESS_AUTH_SALT=..., -e WORDPRESS_SECURE_AUTH_SALT=..., -e WORDPRESS_LOGGED_IN_SALT=..., -e WORDPRESS_NONCE_SALT=... (default to unique random SHA1s, but only if other environment variable configuration is provided)
- -e WORDPRESS_DEBUG=1 (defaults to disabled, non-empty value will enable WP_DEBUG in wp-config.php)
- -e WORDPRESS_CONFIG_EXTRA=... (defaults to nothing, non-empty value will be embedded verbatim inside wp-config.php -- especially useful for applying extra configuration values this image does not provide by default such as WP_ALLOW_MULTISITE; see docker-library/wordpress#142 for more details) \

If the WORDPRESS_DB_NAME specified does not already exist on the given MySQL server, it will be created automatically upon startup of the wordpress container, provided that the WORDPRESS_DB_USER specified has the necessary permissions to create it.


## How to use
### Simple run without DB and volumes

```
docker run -d -p 80:80 w4d00m/wordpress-http2
```

### Run via docker-compose with db (MySQL 5.7) without https.

Change values in **{}** brakets to own!
```
version: '3.2'
services:
  wordpress:
    container_name: wordpress
    image: w4d00m/wordpress-http2
    restart: always
    depends_on:
      - mysql
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: {your_wordpress_database_user}
      WORDPRESS_DB_PASSWORD: {your_wordpress_database_password}
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html
      
  mysql:
    container_name: wordpress_db
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_ROOT_PASSWORD: {your_wordpress_database_user}
      MYSQL_USER: {your_wordpress_database_user}
      MYSQL_PASSWORD: {your_wordpress_database_password} 
    volumes:
      - dbdata:/var/lib/mysql

volumes:
  dbdata:
  wordpress:
```

### Run via docker-compose with db (MySQL 5.7) with https.

```
version: '3.2'
services:
  mysql:
    container_name: wordpress_db
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_ROOT_PASSWORD: {your_wordpress_database_user}
      MYSQL_USER: {your_wordpress_database_user}
      MYSQL_PASSWORD: {your_wordpress_database_password} 
    volumes:
      - dbdata:/var/lib/mysql

  wordpress:
    container_name: wordpress
    image: w4d00m/wordpress-http2
    restart: always
    depends_on:
      - mysql
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: {your_wordpress_database_user}
      WORDPRESS_DB_PASSWORD: {your_wordpress_database_password}
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html
      - certbot-etc:/etc/letsencrypt
      - ./apache:/etc/apache2/sites-enabled
      
    certbot:
    depends_on:
      - wordpress
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - wordpress:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email {your_email} --agree-tos --no-eff-email --force-renewal -d {your_domain_here}
volumes:
  dbdata:
  wordpress:
  certbot:
```

1. Run `docker-compose up -d` and check `docker-compose ps` certbot service status it must have **exit status 0**
2. Rewrite the **./apache/000-default.conf** with next:
   ```
	<VirtualHost *:80>
	        ServerName {your_domain_here}
	        ServerAdmin me@yourdomainherec.om
	        DocumentRoot /var/www/html
	       # Redirect permanent / https://{your_domain_here}/
	</VirtualHost>
	<VirtualHost *:443>
	        ServerName {your_domain_here}
	        ServerAdmin me@yourdomainhere.com
	        DocumentRoot /var/www/html
	        Protocols h2 http/1.1    
	        SSLEngine On
	        SSLCertificateFile /etc/letsencrypt/live/{your_domain_here}/cert.pem
	        SSLCertificateKeyFile /etc/letsencrypt/live/{your_domain_here}/privkey.pem
		SSLCertificateChainFile /etc/letsencrypt/live/{your_domain_here}/chain.pem 
	</VirtualHost>
	SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
	SSLCipherSuite ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS
	SSLHonorCipherOrder on
	SSLCompression off
	SSLSessionTickets off
	SSLUseStapling on
	SSLStaplingResponderTimeout 5
	SSLStaplingReturnResponderErrors off
	SSLStaplingCache shmcb:/var/run/ocsp(128000)
   ```

	**P.S. Don't forgret change values in {} brakets to own!**

3. Restart wordpress contrainer: `docker-compose restart wordpress`
4. If the website work fine uncomment `Redirect parament` for redirect from http to https.
5. Add this script to cron for ssl renew:

	```
	#!/bin/bash
	COMPOSE="/usr/local/bin/docker-compose --no-ansi"
	DOCKER="/usr/bin/docker"
	cd {full_path_to_folder_with_docker-compose.yml}
	$COMPOSE run certbot renew --dry-run && $COMPOSE kill -s SIGHUP wordpress
	$DOCKER system prune -af
	```
