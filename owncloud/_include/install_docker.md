[//]: # (Primary sources)
[//]: # (https://doc.owncloud.org/server/10.4/admin_manual/installation/deployment_recommendations.html#scenario-1-small-workgroups-and-departments)
[//]: # (https://doc.owncloud.org/server/10.4/admin_manual/installation/docker/)

## System Requirements

A machine with:

* Docker

* 100GB storage

* 128 MB RAM

[//]: # (Storage: https://doc.owncloud.org/server/10.4/admin_manual/installation/deployment_recommendations.html#scenario-1-small-workgroups-and-departments)
[//]: # (RAM: https://doc.owncloud.org/server/10.4/admin_manual/installation/system_requirements.html#memory-requirements)

## Installing and Configuring your ownCloud Server

1. Create and run the following script.

        # Create a new project directory
        mkdir owncloud-docker-server

        cd owncloud-docker-server

        # Copy docker-compose.yml from the GitHub repository
        wget https://raw.githubusercontent.com/owncloud/docs/master/modules/admin_manual/examples/installation/docker/docker-compose.yml

        # Create the environment configuration file
        cat << EOF > .env
        OWNCLOUD_VERSION=10.4
        OWNCLOUD_DOMAIN=localhost
        ADMIN_USERNAME=admin
        ADMIN_PASSWORD=admin
        HTTP_PORT=8080
        EOF

        # Build and start the container
        docker-compose up -d

1. When the script is completed, confirm that all the containers are running.

        docker-compose ps

    If the ownCloud server, database, and Redis containers are running, the command will return something like this:

        Name                Command                       State             Ports
        __________________________________________________________________________________________
        server_db_1         /usr/bin/entrypoint/bin/s …   Up                3306/tcp
        server_owncloud_1   /usr/local/bin/entrypoint …   Up                0.0.0.0:8080->8080/tcp
        server_redis_1      /bin/s6-svscan /etc/s6        Up                6379/tcp

1. Wait a few minutes to allow the the ownCloud server to finish initializing all its capabilities. To verify if ownCloud is still initializing:

        docker-compose logs --follow owncloud

    If you see a significant amount of information logging to the console, then we recommend that you wait until you see the logging slow down before you try to connect to ownCloud from your Web browser.

You just installed your ownCloud server!

In your Web browser navigate to `http://localhost:8080` and then log in using the admin username and password that you used above.

### Q&A

* Q: Which web browsers are supported?

  A: You can connect to your ownCloud server from any of the following Web browsers:

  * Edge (current version on Windows 10)

  * IE11 or newer (except Compatibility Mode)

  * Firefox 60 ESR or newer

  * Chrome 66 or newer

  * Safari 10 or newer

* Q: How do I stop the server?

  A: If you're done using the server, you can:

  * Stop the containers

        docker-compose stop

  * Stop and remove the containers along with the related networks, images, and volumes

        docker-compose down --rmi all --volumes

* Q: I'm having issues logging in to the registry; what can I do to fix them?

  A: Make sure the .docker file is in your home directory. If you installed Docker via snap, create a symbolic link to your home directory with the following command:

        ln -sf snap/docker/nnn/.docker

  where `nnn` is the version you are using. For example, if you are using version 384:
  
        ln -sf snap/docker/384/.docker
