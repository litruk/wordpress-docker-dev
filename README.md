# Wordpress Docker images for Plugin, Theme and MultiSite Development

This image is intended for *local development only*. It runs Nginx and PHP-FPM as the *root user* so that Docker volumes mounted at /var/www/html can be written by the server processes.

This is totally awesome for local theme and plugin development, and the worst of all possible worlds if it happens to end up on a real server, so don't. Just don't. :simple_smile:

## Usage

For more detail, see this [initial blog post](http://goldsounds.com/archives/2015/04/06/quick-and-easy-wordpress-development-using-docker/) and the follow up where I [enable multi-site](http://goldsounds.com/archives/2015/04/16/docker-for-wordpress-multisite-development/).

I use this image with [Docker Compose](https://docs.docker.com/compose/), thusly:

0. Install Docker and Docker Compose. Instructions will vary depending on your system.

1. Save the following in a repo somewhere as docker-compose.yml:

```yaml
wordpress:
  image: gravityrail/wordpress
  volumes:
    - src:/var/www/html
  links:
    - db:mysql
  ports:
    - 8080:80

db:
  image: mariadb
  environment:
    MYSQL_ROOT_PASSWORD: example
```

2. Create a directory call "src" in the same directory as docker-compose.yml

3. Run `docker-compose up`. This will boot the WordPress and MariaDB docker images, install WordPress to your "src" directory, and run the server process.

4. *OS X only*: run `boot2docker ip` to find out the IP address of your Docker server.

5. In your web browser, open http://(localhost or docker IP):8080. Congratulations, you should see a WordPress install screen!

You can now install themes and plugins and mess with WordPress as much as you like!

## Scripting the container with wp-cli

You can run the "[wp](http://wp-cli.org/)" cli command as long as you include the "--allow-root" option, for example:

```bash
docker-compose run wordpress wp --allow-root plugin install hello-dolly
```

Or, if you want to create a new plugin, try this:

```bash
docker-compose run wordpress wp --allow-root scaffold plugin my_super_plugin --plugin_name="My Super Plugin" 
```

## Enable MultiSite

The magical, wp-cli way:

```bash
docker-compose run wordpress wp --allow-root core multisite-convert --title="My Blog Network"
```

----

The boring old manual way:

Add this line above the line `/* That's all, stop editing! Happy blogging. */` in wp-config.php:

```php
define( 'WP_ALLOW_MULTISITE', true );
```

Then continue through the installation as normal, and head to "Tools -> Network Setup", and follow all the instructions except those describing .htaccess and Rewrite (those are Apache features, and this image is preconfigured for MultiSite using Nginx). One caveat is that I use "folder" rather than "subdomain" MultiSite, for ease of use. I leave the necessary DNS shenanigans for local subdomain development as an exercise for the reader :simple_smile:

## Credits

All credit must go to https://github.com/docker-library/wordpress, from which this code was shamelessly and inexpertly copied.