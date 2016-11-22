================
Using This Image
================

There are two ways to use this image out-of-the-box:

1. Standalone
2. ZEO cluster

Standalone
----------

.. code-block:: shell

    docker run -p 8080:8080 plone

Now, ask for http://localhost:8080/ in your workstation web browser,
and add a Plone site.

ZEO Cluster
-----------

Start `ZEO` server

.. code-block:: shell

    docker run --name=zeo plone zeoserver

Start 2 Plone clients

.. code-block:: shell

    docker run --link=zeo -e ZEO_ADDRESS=zeo:8100 -p 8081:8080 plone
    docker run --link=zeo -e ZEO_ADDRESS=zeo:8100 -p 8082:8080 plone

Now, open http://localhost:8080/ in your browser.

If you already have a Plone site within ZEO database, click on `View your Plone site`,
otherwise add a new one.

Debug Mode
----------

You can also start Plone in debug mode (fg) by running

.. code-block:: shell

    docker run -p 8080:8080 plone fg

Still, this will not allow you to add `pdb` breakpoints. For this, you'll have
to run Plone inside container like

.. code-block:: shell

    docker run -it -p 8080:8080 plone bash
    bin/instance fg

Add-ons
-------
You can easily test new or existing Plone add-ons by passing them via `PLONE_ADDONS`
environment variable

.. code-block:: shell

    docker run -p 8080:8080 -e PLONE_ADDONS="Products.PloneFormGen eea.facetednavigation" plone fg

The same way as above you can pass `PLONE_ZCML` environment variable to include
custom ZCML files or `PLONE_DEVELOP` environment variable to develop new or
existing Plone add-ons

.. code-block:: shell

    docker run -p 8080:8080 \
                 -e PLONE_ADDONS="plone.theme.winter" \
                 -e PLONE_DEVELOP="src/plone.theme.winter" \
                 -v $(pwd)/src:/plone/instance/src \
                 plone fg

Make sure that you have your Plone add-on code at `src/plone.theme.winter` and
that Plone user inside Docker container (`uid: 500`) has the rights to read/write there.

Running unit tests

.. code-block:: shell

    docker run --rm -e PLONE_ADDONS="eea.facetednavigation" \
             plone \
             bin/test -v -vv -s eea.facetednavigation

.. note::

  Please note that passing `BUILDOUT_` environment variables will slow down
  container creation as a buildout re-run inside container is triggered.
  Thus, we strongly recommend to use this only for testing or development purpose.
  For production use, create a new image as described in the next section.

Extending This Image
--------------------

In order to run Plone with your custom theme or Plone Add-ons, you'll have to
build another image based on this one. For this, you'll need to create two files,
`site.cfg` which is a `zc.buildout <https://pypi.python.org/pypi/zc.buildout/2.5.0>`_
configuration file, and `Dockerfile <https://docs.docker.com/engine/reference/builder/>`_
which is the Docker recipe for your image

site.cfg
~~~~~~~~

.. code-block:: cfg

    [buildout]
    extends = buildout.cfg
    eggs += plone.awsome.addon

Dockerfile
~~~~~~~~~~

.. code-block:: dockerfile

    FROM plone:5

    COPY site.cfg /plone/instance/
    RUN bin/buildout -c site.cfg

Build your custom Plone image

.. code-block:: shell

    docker build -t custom-plone-image .

Run it

.. code-block:: shell

    docker run -p 8080:8080 custom-plone-image

Test it at http://localhost:8080

Environment Variables
---------------------

The Plone image uses several environment variable that allow to specify a more specific setup.

* `PLONE_ADDONS`, `ADDONS` - Customize Plone via Plone add-ons using this environment variable (former `BUILDOUT_EGGS`)
* `PLONE_ZCML`, `ZCML` - Include custom Plone add-ons ZCML files (former `BUILDOUT_ZCML`)
* `PLONE_DEVELOP`, `DEVELOP` - Develop new or existing Plone add-ons (former `BUILDOUT_DEVELOP`)
* `ZEO_ADDRESS` - This environment variable allows you to run Plone image as a ZEO client.
* `ZEO_READ_ONLY` - Run Plone as a read-only ZEO client. Defaults to `off`.
* `ZEO_CLIENT_READ_ONLY_FALLBACK` - A flag indicating whether a read-only remote storage should be acceptable as a fall-back when no writable storages are available. Defaults to `false`.
* `ZEO_SHARED_BLOB_DIR` - Set this to on if the ZEO server and the instance have access to the same directory. Defaults to `off`.
* `ZEO_STORAGE` - Set the storage number of the ZEO storage. Defaults to `1`.
* `ZEO_CLIENT_CACHE_SIZE` - Set the size of the ZEO client cache. Defaults to `128MB`.
* `ZEO_PACK_KEEP_OLD` - Can be set to false to disable the creation of `*.fs.old` files before the pack is run. Defaults to true.
* `HEALTH_CHECK_TIMEOUT` - Time in seconds to wait until health check starts. Defaults to `1` second.
* `HEALTH_CHECK_INTERVAL` - Interval in seconds to check that the Zope application is still healthy. Defaults to `1` second.


.. _data_store:

Where To Store Data
--------------------

.. note::

  There are several ways to store data used by applications that run in
  Docker containers. We encourage users of the `plone` images to familiarize
  themselves with the options available.

The Docker documentation is a good starting point for understanding the different
storage options and variations, and there are multiple blog and forum postings
that discuss and give advice in this area.

Data Volumes (suitable for production use)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let Docker manage the storage of your database data
`by writing the database files to disk on the host system using its own internal volume management <https://docs.docker.com/engine/tutorials/dockervolumes/>`_.
The advantages of this approach is that you can deploy your Plone stack anywhere,
without having to prepare hosts in advance or care about read/write permission
or SELinux `(Security-Enhanced Linux) <https://en.wikipedia.org/wiki/Security-Enhanced_Linux>`_ policy rules. The downside is that the files may be hard to locate
for tools and applications that run directly on the host system, i.e. outside containers.

* Use data volumes with Plone

.. code-block:: shell

    docker run --name plone \
                 --volume=plone-data:/data \
                 -p 8080:8080 \
                 plone

Or with `Docker Compose <https://docs.docker.com/compose/>`_

* Add docker-compose.yml file

.. code-block:: yaml

    plone:
      image: plone
      volumes:
      - plone-data:/data
      ports:
      - "8080:8080"

* Start Plone stack

.. code-block:: shell

    docker-compose up


Mount host directories as data volumes (suitable for development use)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create data directories on the host system (outside the container) and
`mount these to a directory visible from inside the container <https://docs.docker.com/engine/tutorials/dockervolumes/#/mount-a-host-directory-as-a-data-volume>`_.
This places the database files in a known location on the host system, and makes
it easy for tools and applications on the host system to access the files.
The downside is that the user needs to make sure that the directory exists,
and that e.g. directory permissions and other security mechanisms
on the host system are set up correctly.

* Create data directories on a suitable volume on your host system, e.g. `/var/local/data/filestorage` and `/var/local/data/blobstorage`
* Start your `plone` container like this

.. code-block:: shell

    docker run -v /var/local/data/filestorage:/data/filestorage -v /var/local/data/blobstorage:/data/blobstorage -d plone

The -v /path/to/filestorage:/data/filestorage part of the command mounts the -v /path/to/filestorage directory from the underlying host system as /data/filestorage inside the container, where Plone will look for/create the Data.fs database file.

The -v /path/to/blobstorage:/data/blobstorage part of the command mounts the -v /path/to/blobstorage directory from the underlying host system as /data/blobstorage where blobs will be stored.

Make sure that Plone has access to read/write within these folders

.. code-block:: shell

    chown -R 500:500 /var/local/data

Note that users on host systems with SELinux enabled may see issues with this.
The current workaround is to assign the relevant SELinux policy type to the
new data directory so that the container will be allowed to access it

.. code-block:: shell

    chcon -Rt svirt_sandbox_file_t /var/local/data
