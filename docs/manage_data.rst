=============
Managing Data
=============

.. note::

    There are several ways to store data used by applications that run in Docker containers.
    We encourage users of to familiarize themselves with the options available.

The `Docker documentation <https://docs.docker.com/engine/tutorials/dockervolumes/>`_ is a good starting point for understanding the different storage options and variations.

Data Volumes
============

Let Docker manage the storage of your database data `by writing the database files to disk on the host system using its own internal volume management <https://docs.docker.com/engine/tutorials/dockervolumes/>`_.

The advantages of this approach is that you can deploy your Plone stack anywhere, without having to prepare hosts in advance or care about read/write permission
or SELinux `(Security-Enhanced Linux) <https://en.wikipedia.org/wiki/Security-Enhanced_Linux>`_ policy rules.

The downside is that the files may be hard to locate for tools and applications that run directly on the host system, i.e. outside containers.

Use data volumes with Plone:

.. code-block:: shell

    docker run --name plone \
               --volume=plone-data:/data \
               -p 8080:8080 \
               plone

Or use `Docker Compose <https://docs.docker.com/compose/>`_.

Add a `docker-compose.yml` file:

.. code-block:: yaml

    plone:
      image: plone
      volumes:
      - plone-data:/data
      ports:
      - "8080:8080"

Start the Plone stack:

.. code-block:: shell

    docker-compose up

Mount Host Directories As Data Volumes
======================================

Create data directories on the host system (outside the container) and `mount these to a directory visible from inside the container <https://docs.docker.com/engine/tutorials/dockervolumes/#/mount-a-host-directory-as-a-data-volume>`_.

This places the database files in a known location on the host system, and makes it easy for tools and applications on the host system to access the files.

The downside is that the user needs to make sure that the directory exists, and that e.g. directory permissions and other security mechanisms
on the host system are set up correctly.

* Create data directories on a suitable volume on your host system, e.g. `/var/local/data/filestorage` and `/var/local/data/blobstorage`.

Start your container:

.. code-block:: shell

    docker run -v /var/local/data/filestorage:/data/filestorage \
               -v /var/local/data/blobstorage:/data/blobstorage \
               -d plone

The ``-v /path/to/filestorage:/data/filestorage`` part of the command mounts the ``/path/to/filestorage`` directory from the underlying host system as ``/data/filestorage`` inside the container, where Plone will look for/create the Data.fs database file.

The ``-v /path/to/blobstorage:/data/blobstorage`` part of the command mounts the ``/path/to/blobstorage`` directory from the underlying host system as ``/data/blobstorage`` where blobs will be stored.

Make sure that Plone has access to read/write within these folders.

.. code-block:: shell

    chown -R 500:500 /var/local/data

Note that users on host systems with SELinux `(Security-Enhanced Linux) <https://en.wikipedia.org/wiki/Security-Enhanced_Linux>`_ enabled may see issues with this.

The current workaround is to assign the relevant SELinux `(Security-Enhanced Linux) <https://en.wikipedia.org/wiki/Security-Enhanced_Linux>`_ policy type to the new data directory so that the container will be allowed to access it.

.. code-block:: shell

    chcon -Rt svirt_sandbox_file_t /var/local/data


