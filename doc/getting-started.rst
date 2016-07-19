Getting Started
===============

| In this chapter, we'll see a step by step guide to install a ready-to-use infrastructure. For the example, the base we will create will be another Clouder.
| Once installed using Clouder is easy, but set up all the infrastructure may take some time. Execute all this tutorial can take around two hours.


Odoo installation
-----------------

| This guide will not cover the Odoo installation in itself, we suggest you read `the installation documentation on the official website <https://www.odoo.com/documentation/8.0/setup/install.html>`_.
| You can also contact us on `www.goclouder.net <https://www.goclouder.net/>`_ to ask for a free instance, it may be easier for you because you obviously can't install your Clouder on a server managed by this same Clouder.
| Due to the extensive use of ssh, Clouder is only compatible with Linux and has only be tested on a Debian distribution.

Once your Odoo installation is ready, download the Clouder modules on `Github <https://github.com/clouder-community/clouder/>`_ and add them in your addons directory, then install the clouder module and clouder_template_odoo (this module will install a lot of template dependencies, like postgres, postfix etc...).

.. image:: images/gettingstarted-moduleslist.png


Clouder configuration
---------------------

The first thing to do is to set the sysadmin email in the Clouder configuration.

.. image:: images/gettingstarted-configuration.png

Then you have to configure the server you want to connect. See the `how to connect a server chapter <connect-server.rst>`_ for more explanation.

.. image:: images/gettingstarted-server.png


The registry container
----------------------

The next thing to do is to generate the images for each applications we'll need, but for this we need a registry container to store theses images.

First, go in the img_registry in the Images menu.

.. image:: images/gettingstarted-registry-image.png

And click on the Build button to generate a new version.

Then go on the container menu and create a new registry container.

.. image:: images/gettingstarted-registry-container.png

When you save it, the registry image will be downloaded, build and deployed as a new container. Like all actions in Clouder, you can read the log to understand which commands was executed.

.. image:: images/gettingstarted-registry-container-log.png

The logs are also visible in the default Odoo output.


Build the images
----------------

Now that we have the registry, we can build the images.

First we need to build the base image, which will be used by all the others. This image update the debian package, install cron and supervisor and some useful packages.

.. image:: images/gettingstarted-base-image.png

Select the registry container, and click on the Build button to generate it.

Then you can generate the others images. For this tutorial, we'll need theses ones :

- img_archive

- img_backup_bup

- img_backup_upload

- img_nginx

- img_bind

- img_postfix

- img_shinken

- img_postgres

- img_odoo

.. image:: images/gettingstarted-odoo-image.png

For each, select the registry container and the version of the base image you want to use, then build it. Some of them will take time, especially shinken which may take around twenty minutes to generate.

See the `Image chapter <images.rst>`_ for more informations about the fields in the image form.


Deploy containers
-----------------

The first containers we'll need to deploy will be the backup and backup-upload containers because theses are containers used by Clouder (like the registry one).

First generate the backup-upload containers (because they are containers without link). Note that backup-upload container shall be in a distant server to protect your backups, but for this tutorial you can let it in the same server.

.. image:: images/gettingstarted-backupupload-container.png

Then, create the backup container. This one require a link to the backup-upload to know where the backup shall be exported.

.. image:: images/gettingstarted-backup-container.png

Now that the system containers are created, we can generate the containers for the subcomponents needed for hosting Odoo. For each, think about adding the backup container in the save tab.

| Postfix for sending and receiving emails. You shall use an smtp provider to make sure your emails are send (mandrill is already configured, just indicate your email and api in the container options).
| You have to map the port 25 of your server to the port 25 of your postfix container to receive emails.

.. image:: images/gettingstarted-postfix-container.png

Bind for resolving domain names. You have to map the port 53 of your server to the port 53 of your Bind container.

.. image:: images/gettingstarted-bind-container.png

Shinken for monitoring. If something goes wrong on your infrastructure an email will be send to the sysadmin email.

Are monitored :

-The server cpu/RAM/disk space.

-If all the containers are accessible through ssh.

-If all bases are accessible at their url.

-If all containers and bases were backuped today.

You have to specify the postfix container which will send the emails.

.. image:: images/gettingstarted-shinken-container.png

| Proxy for redirecting each url to the correct container.
| You have to map the ports 80 and 443 of your server to the port 80 and 443 of your proxy container.

.. image:: images/gettingstarted-proxy-container.png

Postgres for managing your databases.

.. image:: images/gettingstarted-postgres-container.png

Finally, you can deploy your Odoo container. Indicate your postfix and postgres container in the links.

.. image:: images/gettingstarted-clouder-container.png

If you want to monitor ssh connexion and container backup, you can add shinken in the links and click on deploy. You can do it anytime, even after creating the container.

For more information about the container fields, you can read the `Containers chapter <containers.rst>`_.


Build the application
---------------------

Next, you need to go on the Clouder application in the Applications menu.

.. image:: images/gettingstarted-application.png

Indicate the archive container and click on the build button. This will download the Odoo files and the Clouder modules.

For more information about the application fields, you can read the `Applications chapter <applications.rst>`_.


Deploy the service
------------------

Now that we have the application, we can deploy it in our Odoo container.

.. image:: images/gettingstarted-service.png

| Don’t forget to indicate the postgres container in the link.
| For more information about the service fields, you can read the `Services chapter <services.rst>`_.


Configure the domain
--------------------

Before we can add our base, we need to add the domain in your Clouder.

.. image:: images/gettingstarted-domain.png

Configure your domain at your domain provider to use your server as the DNS server (take care, the change may take days to propagate). This is optional, you can continue to use the DNS of your provider but in this case you’ll have to add manually the record each time you create a base in your Clouder.

For more information about the domain fields, you can read the `Domains and Bases chapter <domains-bases.rst>`_.


Add the base
------------

Finally, we can configure the base.

.. image:: images/gettingstarted-base.png

Don’t forget to configure the bind/shinken/proxy/postfix links and the backup container. The database will be automatically created, your Clouder will be configured and the proxy will redirect the url to your Odoo container.

.. image:: images/gettingstarted-result.png

For more information about the base fields, you can read the `Domains and Bases chapter <domains-bases.rst>`_.

Go to your shinken instance to see the health of your infrastructure in real time (Default login : admin / admin)

.. image:: images/gettingstarted-shinken.png

Congratulations! You can now easily create another base or deploy any other application you can find in the clouder_template_* modules, or even create your own images and applications.

| If you need any assistance you can contact us for professional services or ask on the forum/mailing-list in `www.goclouder.net <https://www.goclouder.net/>`_.
| If you want to report a bug or contribute, go to the github repository `https://github.com/clouder-community/clouder <https://github.com/clouder-community/clouder/>`_.
