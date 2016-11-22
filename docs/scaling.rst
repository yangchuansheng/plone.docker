=======
Scaling
=======

Scaling
-------

In order to scale up and down the number of ZEO clients you'll need
orchestration tools like `docker-compose <https://docs.docker.com/compose/install/>`_

Bellow is a `docker-compose.yml` example with ZEO server and Plone
instance configured as a ZEO client::

  haproxy:
    image: eeacms/haproxy
    ports:
    - 8080:5000
    - 1936:1936
    links:
    - plone
    environment:
    - BACKENDS_PORT=8080
    - SERVICE_NAMES=plone

  plone:
    image: plone
    links:
    - zeoserver
    environment:
    - ZEO_ADDRESS=zeoserver:8100

  zeoserver:
    image: plone
    command: zeoserver

Start cluster::

  $ docker-compose up -d

Scale the number of ZEO clients::

  $ docker-compose scale plone=4

Now, open http://localhost:8080 in your workstation web browser. To see the
HAProxy backend health, go to http://localhost:1936 Default user: `admin/admin`
