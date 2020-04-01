[//]: # (Sources)
[//]: # (https://doc.owncloud.com/server/admin_manual/installation/changing_the_web_route.html)
[//]: # (https://stackoverflow.com/questions/3940909/configure-apache-to-listen-on-port-other-than-80)
[//]: # (https://www.tecmint.com/change-apache-port-in-linux/)

## Customizing your ownCloud URL 

### Enabling Service at the Root Level

> Before you make this change, make sure you don't want to offer other virtual hosts on this server. After you make this change, your users will be able to access only the ownCloud virtual host.

1. Edit `/etc/apache2/sites-enabled/owncloud.conf` and change the `Alias` entry:

        Alias / "/var/www/owncloud/"

1. Edit `/var/www/owncloud/config/config.php` and change the `overwrite.cli.url` parameter:

        'overwrite.cli.url' => 'http://localhost/',

1. Restart Apache.

        sudo service apache2 restart

### Enabling Service through a Different Port

For example, if you want to provide service through port `8080`:

1. Edit `/etc/apache2/ports.conf`, add the following line:

        Listen 8080

1. Edit `/etc/apache2/sites-enabled/000-default.conf` and change the `VirtualHost` entry to be:

        <VirtualHost *: 8080>

1. Restart Apache.

        sudo service apache2 restart