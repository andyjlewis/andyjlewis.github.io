[//]: # (Primary sources)
[//]: # (https://doc.owncloud.org/server/10.4/admin_manual/installation/deployment_recommendations.html#scenario-1-small-workgroups-and-departments)
[//]: # (https://doc.owncloud.org/server/10.4/admin_manual/installation/ubuntu_18_04.html)

## System Requirements

A machine with:

* A fresh install of the Ubuntu 18.04 LTS operating system with SSH enabled

* 100GB storage

* 128 MB RAM

[//]: # (OS: https://doc.owncloud.org/server/10.4/admin_manual/installation/system_requirements.html#officially-recommended-environment)
[//]: # (Storage: https://doc.owncloud.org/server/10.4/admin_manual/installation/deployment_recommendations.html#scenario-1-small-workgroups-and-departments)
[//]: # (RAM: https://doc.owncloud.org/server/10.4/admin_manual/installation/system_requirements.html#memory-requirements)

In the following sections we'll guide you through installing the following software: 

* PHP 7.3

* Apache 2.4 web server with prefork and mod_php

* MySQL 8 or newer database server

* LDAP server for authentication

## Installing and Configuring your ownCloud Server

Connect to your machine as the root user.

### Prepare Your Machine

1. Make sure that all of the installed packages are updated, and that PHP is available in the APT repository.

       apt update && \ 
          apt upgrade -y


1. Create a helper script to simplify running the ownCloud console (occ) command that you'll use in the sections below.

       FILE="/usr/local/bin/occ"
       /bin/cat <<EOM >$FILE
       #! /bin/bash

       cd /var/www/owncloud
       sudo -u www-data /usr/bin/php /var/www/owncloud/occ "\$@"
       EOM

1. Make the helper script executable.

       chmod +x /usr/local/bin/occ

1. Install the required packages.

       apt install -y \
          apache2 \
          libapache2-mod-php \
          mariadb-server \
          openssl \
          php-imagick php-common php-curl \
          php-gd php-imap php-intl \
          php-json php-mbstring php-mysql \
          php-ssh2 php-xml php-zip \
          php-apcu php-redis redis-server \
          wget

1. Install the recommended packages.

        apt install -y \
         ssh bzip2 rsync curl jq \
         inetutils-ping smbclient\
         php-smbclient coreutils php-ldap

[//]: # (I would do research and possibly explain a little more about why these are recommended)

### Configure the Apache Web Server

1. Change the document root.

        sed -i "s#html#owncloud#" \
            /etc/apache2/sites-available/000-default.conf

        service apache2 restart

1. Create a virtual host configuration.

        FILE="/etc/apache2/sites-available/owncloud.conf"
        sudo /bin/cat <<EOM >$FILE
        Alias /owncloud "/var/www/owncloud"

        <Directory /var/www/owncloud>
          Options +FollowSymlinks
          AllowOverride All

         <IfModule mod_dav.c>
           Dav off
         </IfModule>

          SetEnv HOME /var/www/owncloud
          SetEnv HTTP_HOME /var/www/owncloud
        </Directory>
        EOM

1. Enable the virtual host configuration.

        a2ensite owncloud.conf
        service apache2 reload


### Configure the Database

    mysql -u root -e "CREATE DATABASE IF NOT EXISTS owncloud; \
    GRANT ALL PRIVILEGES ON owncloud.* \
      TO owncloud@localhost \
      IDENTIFIED BY 'password'";

### Enable the Recommended Apache Modules

    echo "Enabling Apache Modules"

    a2enmod dir env headers mime rewrite setenvif
    service apache2 reload

### Set up ownCloud

1. Download ownCloud.

        cd /var/www/
        wget https://download.owncloud.org/community/owncloud-10.2.1.tar.bz2 && \
        tar -xjf owncloud-10.2.1.tar.bz2 && \
        chown -R www-data. owncloud

1. Install ownCloud.

        occ maintenance:install \
            --database "mysql" \
            --database-name "owncloud" \
            --database-user "owncloud" \
            --database-pass "password" \
            --admin-user "admin" \
            --admin-pass "admin"

1. Configure ownCloud's trusted domains.

        myip=$(hostname -I|cut -f1 -d ' ')
        occ config:system:set trusted_domains 1 --value="$myip"

1. Set up a Cron job.

        echo "*/15  *  *  *  * /usr/bin/php /path/to/your/owncloud/occ system:cron" \
        > /var/spool/cron/crontabs/www-data
        chown www-data.crontab /var/spool/cron/crontabs/www-data
        chmod 0600 /var/spool/cron/crontabs/www-data

    If you need to sync your users from an LDAP or Active Directory Server, then add this additional Cron job.

        echo "*/15  *  *  *  * /usr/bin/php \
            /path/to/your/owncloud/occ system:cron" > /var/spool/cron/crontabs/www-data
        chown www-data.crontab  /var/spool/cron/crontabs/www-data
        chmod 0600  /var/spool/cron/crontabs/www-data

1. Configure caching and file locking.

        occ config:system:set \
            memcache.local \
            --value '\OC\Memcache\APCu'

        occ config:system:set \
            memcache.locking \
            --value '\OC\Memcache\Redis'

        occ config:system:set \
            redis \
            --value '{"host": "127.0.0.1", "port": "6379"}' \
            --type json

1. Configure log rotation.

        FILE="/etc/logrotate.d/owncloud"
        sudo /bin/cat <<EOM >$FILE
        /var/www/owncloud/data/owncloud.log {
            size 10M
            rotate 12
            copytruncate
            missingok
            compress
            compresscmd /bin/gzip
        }
        EOM

1. Finalize the installation.

        FILE="/usr/local/bin/ocpermissions"

        /bin/cat <<EOM >$FILE
        #!/bin/bash

        ocpath="{install-directory}"
        datadir="{install-directory}/data"
        htuser="{webserver-user}"
        htgroup="{webserver-group}"
        rootuser="root"

        printf "Creating any missing directories"
        sudo -u "${htuser}" mkdir -p "$ocpath/assets"
        sudo -u "${htuser}" mkdir -p "$ocpath/updater"
        sudo -u "${htuser}" mkdir -p "$datadir"

        printf "Update file and directory permissions"
        sudo find "${ocpath}/" -type f -print0 | xargs -0 chmod 0640
        sudo find "${ocpath}/" -type d -print0 | xargs -0 chmod 0750

        printf "Set web server user and group as ownCloud directory user and group"
        sudo chown -R "${rootuser}:${htgroup}" "${ocpath}/"
        sudo chown -R "${htuser}:${htgroup}" "${ocpath}/apps/"
        sudo chown -R "${htuser}:${htgroup}" "${ocpath}/apps-external/"
        sudo chown -R "${htuser}:${htgroup}" "${ocpath}/assets/"
        sudo chown -R "${htuser}:${htgroup}" "${ocpath}/config/"
        sudo chown -R "${htuser}:${htgroup}" "${datadir}"
        sudo chown -R "${htuser}:${htgroup}" "${ocpath}/updater/"
        sudo chmod +x "${ocpath}/occ"

        printf "Set web server user and group as .htaccess user and group"
        if [ -f "${ocpath}/.htaccess" ]; then
            sudo chmod 0644 "${ocpath}/.htaccess"
            sudo chown "${rootuser}:${htgroup}" "${ocpath}/.htaccess"
        fi

        if [ -f "${datadir}/.htaccess" ]; then
            sudo chmod 0644 "${datadir}/.htaccess"
            sudo chown "${rootuser}:${htgroup}" "${datadir}/.htaccess"
        fi

        EOM

        # Make the script executable
        sudo chmod +x /usr/local/bin/ocpermissions

        ocpermissions

You just installed your ownCloud server!

### Q&A

* Q: How Much Scale Can the Above Deployment Support?

  A: The above deployment can support up to 150 users with up to 10TB of data.

* Q: What Kind of Availability Can the Above Deployment Offer?

  A: The availability depends on the file system and backup approach you use:

    - On a Btrfs file system, you can achieve zero-downtime backups by using snapshots. (However, component failure will still cause interruption of service.)

    - On other kinds of file systems, you can use nightly backup schemes to limit service interruptions to less busy times.

* Q: What SMB limitations are there When Using Ubuntu 18.04?

  A: Ubuntu 18.04 includes smbclient 4.7.6, which has a known limitation of only using version 1 of the SMB protocol.

* Q: What are prefork and mod_php?

  A: [prefork and mod_php](https://doc.owncloud.org/server/10.4/admin_manual/installation/manual_installation.html#multi-processing-module-mpm)

* Q: Where can I Learn More about ownClould Console (occ) Commands?

  A: [Using the occ Command](https://doc.owncloud.org/server/10.4/admin_manual/configuration/server/occ_command.html)