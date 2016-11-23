=====
Usage
=====

There are two ways to use this image out-of-the-box:

1. Standalone
2. ZEO Cluster

The standalone method is easier for development, testing or demoing, while the clustered (ZEO) setup will take advantage of multi-core CPUs and is recommended for a production deployment.

Standalone
----------

.. code-block:: shell

    docker run -p 8080:8080 plone

Open http://localhost:8080/ in your browser, and add a Plone site.

ZEO Cluster
-----------

Start `ZEO` server

.. code-block:: shell

    docker run --name=zeo plone/zeoserver


Start 2 Plone clients

.. code-block:: shell

    docker run --link=zeo -e ZEO_ADDRESS=zeo:8100 -p 8081:8080 plone
    docker run --link=zeo -e ZEO_ADDRESS=zeo:8100 -p 8082:8080 plone

Open http://localhost:8080/ in your browser.

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

Add-Ons
-------
You can easily test new or existing Plone add-ons by passing them via ``PLONE_ADDONS``
:ref:`anchor_environment_variables`.

.. code-block:: shell

    docker run -p 8080:8080 -e PLONE_ADDONS="Products.PloneFormGen eea.facetednavigation" plone fg

The same way as above you can pass ``PLONE_ZCML`` environment variable to include
custom ZCML files or ``PLONE_DEVELOP`` environment variable to develop new or
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

.. _anchor_environment_variables:

Environment Variables
---------------------

The Plone image uses several environment variables which allows to specify a more specific setup.

If you want to know more about them in general, please read their `official documentation <https://docs.docker.com/engine/reference/run/#/env-environment-variables>`_.

* ``PLONE_ADDONS``, ``ADDONS`` - Customize Plone via Plone add-ons using this environment variable (former ``BUILDOUT_EGGS``)
* ``PLONE_ZCML``, ``ZCML`` - Include custom Plone add-ons ZCML files (former ``BUILDOUT_ZCML``)
* ``PLONE_DEVELOP``, ``DEVELOP`` - Develop new or existing Plone add-ons (former ``BUILDOUT_DEVELOP``)
* ``ZEO_ADDRESS`` - This environment variable allows you to run Plone image as a ZEO client.
* ``ZEO_READ_ONLY`` - Run Plone as a read-only ZEO client. Defaults to ``off``.
* ``ZEO_CLIENT_READ_ONLY_FALLBACK`` - A flag indicating whether a read-only remote storage should be acceptable as a fall-back when no writable storages are available. Defaults to ``false``.
* ``ZEO_SHARED_BLOB_DIR`` - Set this to on if the ZEO server and the instance have access to the same directory. Defaults to ``off``.
* ``ZEO_STORAGE`` - Set the storage number of the ZEO storage. Defaults to ``1``.
* ``ZEO_CLIENT_CACHE_SIZE`` - Set the size of the ZEO client cache. Defaults to ``128MB``.
* ``ZEO_PACK_KEEP_OLD`` - Can be set to false to disable the creation of ``*.fs.old`` files before the pack is run. Defaults to true.
* ``HEALTH_CHECK_TIMEOUT`` - Time in seconds to wait until health check starts. Defaults to ``1`` second.
* ``HEALTH_CHECK_INTERVAL`` - Interval in seconds to check that the Zope application is still healthy. Defaults to ``1`` second.


