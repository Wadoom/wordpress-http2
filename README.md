# wordpress-http2
The WordPress 5.5 docker image with apache2.4 (http2). The image based on Ubuntu 20.04, php-fpm-7.4. \
Main target its create image with the possibility to use Apache2 on http2 protocol. The big part of the code was taken from official docker file wordpress:5.5 and php7.3:apache/fpm.

## ENV
The image has same environment variables as official wordpress docker image [Wordpress Docker Hub](https://hub.docker.com/_/wordpress) \
The following environment variables are also honored for configuring your WordPress instance:

⋅⋅⋅-e WORDPRESS_DB_HOST=...
⋅⋅⋅-e WORDPRESS_DB_USER=...
⋅⋅⋅-e WORDPRESS_DB_PASSWORD=...
⋅⋅⋅-e WORDPRESS_DB_NAME=...
⋅⋅⋅-e WORDPRESS_TABLE_PREFIX=...
⋅⋅⋅-e WORDPRESS_AUTH_KEY=..., -e WORDPRESS_SECURE_AUTH_KEY=..., -e WORDPRESS_LOGGED_IN_KEY=..., -e WORDPRESS_NONCE_KEY=..., -e WORDPRESS_AUTH_SALT=..., -e WORDPRESS_SECURE_AUTH_SALT=..., -e WORDPRESS_LOGGED_IN_SALT=..., -e WORDPRESS_NONCE_SALT=... (default to unique random SHA1s, but only if other environment variable configuration is provided)
⋅⋅⋅-e WORDPRESS_DEBUG=1 (defaults to disabled, non-empty value will enable WP_DEBUG in wp-config.php)
⋅⋅⋅-e WORDPRESS_CONFIG_EXTRA=... (defaults to nothing, non-empty value will be embedded verbatim inside wp-config.php -- especially useful for applying extra configuration values this image does not provide by default such as WP_ALLOW_MULTISITE; see docker-library/wordpress#142 for more details) \
If the WORDPRESS_DB_NAME specified does not already exist on the given MySQL server, it will be created automatically upon startup of the wordpress container, provided that the WORDPRESS_DB_USER specified has the necessary permissions to create it.


##How to use
### Simple run without DB and volumes

```
docker run -d -p 80:80 w4d00m/wordpress-http2
```