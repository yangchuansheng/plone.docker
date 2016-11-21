===============
Plone On Docker
===============

.. toctree::
   :hidden:
   :maxdepth: 2

   usage

.. admonition:: Description

    Use Docker to run Plone.

You can run Plone with `Docker <https://www.docker.com/>`_. This is a great way, for testing add-ons, continuous integration  and giving demos or training sessions.

If you have the knowledge, you can also use Docker for development or for running Plone in production.

**This is not advised for beginner and people who are new to Docker !**

.. note::

    Running Plone on Docker in production requires solid knowledge of Docker, you should be familiar with monitoring, logging, backups and hosting docker-based applications.

    - Use on your own risk!
    - The Plone Foundation nor the Plone Community are responsible for any data loss!



.. warning::

    If you start the container without any configuration for data storage, you will run the container without persistent data storage.

    That means if your container fails or you start,stop or restart all data will be gone.

    In order to avoid that please make sure sure that you read our docs about :ref:`data_store`.
