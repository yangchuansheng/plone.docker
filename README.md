# Plone

[Plone](https://plone.org) is a free and open source content management system built on top of the Zope application server.


## Features

- Images for Plone 5 and Plone 4
- Choose between [Debian](https://www.debian.org/) or [Alpine (unstable)](http://www.alpinelinux.org/) based images

### Supported tags and respective `Dockerfile` links

- [`5.0.6`, `5.0`, `5`, `latest` (*5.0.6/Dockerfile*)](https://github.com/plone/plone.docker/blob/master/5.0/5.0.6/debian/Dockerfile)
- [`4.3.11`, `4.3`, `4` (*4.3.11/Dockerfile*)](https://github.com/plone/plone.docker/blob/master/4.3/4.3.11/debian/Dockerfile)
- [`5.0.6-alpine`, `5.0-alpine`, `5-alpine` (*5.0.6/alpine/Dockerfile*)](https://github.com/plone/plone.docker/blob/master/5.0/5.0.6/alpine/Dockerfile)
- [`4.3.11-alpine`, `4.3-alpine`, `4-alpine` (*4.3.11/alpine/Dockerfile*)](https://github.com/plone/plone.docker/blob/master/4.3/4.3.11/alpine/Dockerfile)

## Usage

### Start a single Plone instance

This will download and start the latest Plone 5 container, based on [Debian](https://www.debian.org/).

```console
$ docker run -p 8080:8080 plone
```

This image includes `EXPOSE 8080` (the Plone port), so standard container linking will make it automatically available to the linked containers. Now you can add a Plone Site at http://localhost:8080 - default Zope user and password are `admin/admin`.

### Start Plone within a ZEO cluster

Start ZEO server

```console
$ docker run --name=zeo plone zeoserver
```

Start 2 Plone clients

```console
$ docker run --link=zeo -e ZEO_ADDRESS=zeo:8100 -p 8081:8080 plone
$ docker run --link=zeo -e ZEO_ADDRESS=zeo:8100 -p 8082:8080 plone
```

### Start Plone in debug mode

You can also start Plone in debug mode (`fg`) by running

```console
$ docker run -p 8080:8080 plone fg
```

For more information on how to extend this image with your own custom settings, adding more add-ons, building it or mounting volumes, please refer to our [documentation](https://github.com/plone/plone.docker/tree/master/docs).

### Supported Environment Variables

The Plone image uses several environment variable that allow to specify a more specific setup.

* `PLONE_ADDONS`, `ADDONS` - Customize Plone via Plone add-ons using this environment variable (former `BUILDOUT_EGGS`)
* `PLONE_ZCML`, `ZCML` - Include custom Plone add-ons ZCML files (former `BUILDOUT_ZCML`)
* `PLONE_DEVELOP`, `DEVELOP` - Develop new or existing Plone add-ons (former `BUILDOUT_DEVELOP`)
* `ZEO_ADDRESS` - This environment variable allows you to run Plone image as a ZEO client.
* `ZEO_READ_ONLY` - Run Plone as a read-only ZEO client. Defaults to `off`.
* `ZEO_CLIENT_READ_ONLY_FALLBACK` - A flag indicating whether a read-only remote storage should be acceptable as a fallback when no writable storages are available. Defaults to `false`.
* `ZEO_SHARED_BLOB_DIR` - Set this to on if the ZEO server and the instance have access to the same directory. Defaults to `off`.
* `ZEO_STORAGE` - Set the storage number of the ZEO storage. Defaults to `1`.
* `ZEO_CLIENT_CACHE_SIZE` - Set the size of the ZEO client cache. Defaults to `128MB`.
* `ZEO_PACK_KEEP_OLD` - Can be set to false to disable the creation of *.fs.old files before the pack is run. Defaults to true.
* `HEALTH_CHECK_TIMEOUT` - Time in seconds to wait until health check starts. Defaults to `1` second.
* `HEALTH_CHECK_INTERVAL` - Interval in seconds to check that the Zope application is still healthy. Defaults to `1` second.

## Documentation

Full documentation for end users can be found in the ["docs"](https://github.com/plone/plone.docker/tree/master/docs) folder.
It is also available online at http://docs.plone.org/


## Contribute


- Issue Tracker: https://github.com/plone/plone.docker/issues
- Source Code: https://github.com/plone/plone.docker
- Documentation: http://docs.plone.org/

## Support


If you are having issues, please let us know at https://community.plone.org

## License

The project is licensed under the GPLv2.
