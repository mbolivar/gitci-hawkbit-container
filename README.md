Run Hawkbit in a Docker Container
=================================

* For info about Hawkbit see: https://github.com/eclipse/hawkbit/

Build the Image
===============

You might want to change some values from the default ones used in the configuration
files.

Look at the [application.properties](./application.properties) file and change:

* Default RabbitMQ username and password (`spring.rabbitmq.username`, `spring.rabbitmq.password`) (if different).
* MariaDB URL, username and password (`spring.datasource.url`, `spring.datasource.username`, `spring.datasource.password`), and the name of the database used.
* MongoDB URI (`spring.data.mongodb`).
* The Hawkbit password and user (`hawkbit.server.ui.demo.password`, `hawkbit.server.ui.demo.user`).

The image can take a while to build.

    docker build -t hawkbit --force-rm .

Run the Container
=================

The container now needs three more containers to run:

* A [MariaDB one](https://store.docker.com/images/1712cb54-62e1-405b-a973-1492552c9bb9) (called `srv-mariadb`)
* A [RabbitMQ one](https://store.docker.com/images/fa7625b4-fdca-4b48-b078-692f6451965a) (called `srv-rabbitmq`)
* A [mongodb one](https://store.docker.com/images/9147d1b7-a686-4e38-8ecd-94a47f5da9cf) (called `srv-mongodb`)

Since the `--link` options is a [deprecated legacy](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/),
it is necessary to create a [dedicated network](https://docs.docker.com/engine/userguide/networking/work-with-networks/) where all the above containers,
plus the Hawkbit one, will run.

Create the `hawkbit-net` network:

    docker network create hawkbit-net

Start a MariaDB container as explained in the [offial container image documentation](https://hub.docker.com/_/mariadb/).

You need to specify the `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE` values,
and those should be reflected in the `application.properties` file.

    docker run -dit --network=hawkbit-net --name srv-mariadb -e MYSQL_ROOT_PASSWORD=ROOT_PASSWORD -e MYSQL_USER=USER_NAME -e MYSQL_PASSWORD=USER_PASSWORD -e MYSQL_DATABASE=DB_NAME mariadb

Run rabbitmq and mongdb:

    docker run -dit --network=hawkbit-net --name srv-mongodb mongo:3.2
    docker run -dit --network=hawkbit-net --name srv-rabbitmq --hostname srv-rabbitmq rabbitmq

When all the containers are up, start the Hawkbit one:

    docker run -dit --network=hawkbit-net --name hawkbit -p 8080:8080 linaroproduct/ -v secrets.properties:/srv/secret/secrets.properties gitci-hawkbit-container

The `secrets.properties` file available in the repository is just an example:
it should match at least the values defined for the Mariadb container before.
The file has to be mounted at `/srv/secrets/secrets.properties`.

If you prefer to use the `--link` mode to connect the various containers, remove
the `--network` argument from the commands above, and on the last one (the one
that runs Hawkbit) include:

    --link srv-mariadb:srv-mariadb --link srv-mongodb:srv-mongodb --link srv-rabbitmq:srv-rabbitmq

Once the container is up and running and the application started (in can take
~40 seconds for the application to start the first time), head to http://localhost:8080/UI
and login with the Hawkbit username and password configured (by default `admin`/`admin`).

From here on:

> Hic sunt dracones
