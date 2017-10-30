# Jena Fuseki 2 docker image

Forked from stain/jena-fuseki

The TAG 3.4.0 runs as root user with a mount point at /fuseki
The TAG 3.4.0b and latest run as a non-root user with a mount point at /var/fuseki_home

* Docker image: [dskow/jena-fuseki](https://hub.docker.com/r/dskow/jena-fuseki/)
* Base images:  [centos](https://hub.docker.com/_/centos/):centos7
* Source: [Dockerfile](https://github.com/dskow/jena-fuseki/Dockerfile), [Apache Jena Fuseki](http://jena.apache.org/download/)

[![Build Status](https://travis-ci.org/dskow/jena-fuseki.svg)](https://travis-ci.org/dskow/jena-fuseki)

[![](https://images.microbadger.com/badges/image/dskow/jena-fuseki.svg)](https://microbadger.com/images/dskow/jena-fuseki "Get your own image badge on microbadger.com")

[![](https://images.microbadger.com/badges/version/dskow/jena-fuseki.svg)](https://microbadger.com/images/dskow/jena-fuseki "Get your own version badge on microbadger.com")


This is a [Docker](https://www.docker.com/) image for running
[Apache Jena Fuseki 2](https://jena.apache.org/documentation/fuseki2/),
which is a [SPARQL 1.1](http://www.w3.org/TR/sparql11-overview/) server with a
web interface, backed by the
[Apache Jena TDB](https://jena.apache.org/documentation/tdb/) RDF triple store.

Feel free to contact the [jena users
list](https://jena.apache.org/help_and_support/) for any questions on using
Jena or Fuseki.

## License

Different licenses apply to files added by different Docker layers:

* dskow/jena-fuseki [Dockerfile](https://github.com/dskow/jena-fuseki): [Apache License, version 2.0](http://www.apache.org/licenses/LICENSE-2.0)
* Apache Jena (`/opt/fuseki` in the image): [Apache License, version 2.0](http://www.apache.org/licenses/LICENSE-2.0)
* OpenJDK (/opt/jdk1.8.0_152 in the image): GPL 2.0 with Classpath exception
* CentOS Linux 7 (rest of /): Free software (GPL and other licenses)
* Tini [Site](https://github.com/krallin/tini): [MIT](https://github.com/krallin/tini/blob/master/LICENSE)

## Use

To try out this image, try:

    docker run dskow/jena-fuseki

The Apache Jena Fuseki should then be available at http://localhost:3030/

To expose Fuseki on a different port, simply modify first part of `-p`: 

    docker run -p 8080:3030 dskow/jena-fuseki

To load RDF graphs, you will need to log in as the `admin` user. To see the
automatically generated admin password, see the output from above, or
use `docker logs` with the name of your container.

Note that the password is only generated on the first run, e.g. when the
volume `/var/fuseki_home` is an empty directory.

You can override the admin-password using the form
`-e ADMIN_PASSWORD=pw123`:

    docker run -e ADMIN_PASSWORD=pw123 dskow/jena-fuseki

To specify Java settings such as the amount of memory to allocate for the
heap (default: 1200 MiB), set the `JVM_ARGS` environment with `-e`:

    docker run -e JVM_ARGS=-Xmx2g dskow/jena-fuseki

## default user

The fuseki container uses user id 1000 and group id 1000 for the file permissions.  The data container directories or host directories should use the same user and group.  The user and group can only be changed with a costum build.

## custom build

Some defaults can be changed by custom building the image with build arguments.

	docker build \
	   --build-arg user=newfuseki \
	   --build-arg uid=1001 \
	   --build-arg group=newfuseki \
	   --build-arg gid=1001 \
	   -t dskow/jena-fuseki .

Here are some other build args: FUSEKI_VERSION_MAJOR, FUSEKI_VERSION_MINOR, FUSEKI_VERSION_PATCH, and FUSEKI_SHA
	
## Data persistence

If the data volume is not defined.  The data is stored in /tmp with the default shiri.ini file and not persisted.  If the data volume exists with the correct permission but the shiro.ini file is missing, a copy of the default file will be placed in it.  If the data volume exists with the correct permissions, the data will be persisted.

Fuseki's data is stored in the Docker volume `/var/fuseki_home` within the container.
Note that unless you use `docker restart` or one of the mechanisms below, data is lost between each run of the jena-fuseki image.

To store the data in a named Docker volume container `fuseki-data`
(recommended), create it first as:

    docker run --name fuseki-data -v /var/home_fuseki busybox

The default owner is root:root.  You will need to change ownership to the proper user id and group id. The id's are from the host machine. This example has user id 1000 and group id 1000

	docker run --rm \
	   --volumes-from fuseki-data ubuntu bash \
	   -c "find /var/fuseki_home | xargs -i chown 1000:1000 {}"
	
Then start fuseki using `--volumes-from`. This allows you to later upgrade the
jena-fuseki docker image without losing the data. The command below also uses
`-d` to start the container in the background.

    docker run -d --name fuseki \
	   --volumes-from fuseki-data dskow/jena-fuseki
	
If you want to store fuseki data in a specified location on the host (e.g. for
disk space or speed requirements), specify it using `-v`:

    docker run -d --name fuseki \
	   -v /ssd/data/fuseki:/var/home_fuseki \
	   dskow/jena-fuseki

Note that the `/var/fuseki_home` volume must only be accessed from a single Fuseki
container at a time.

To check the logs for the container you gave `--name fuseki`, use:

    docker logs fuseki

To stop the named container, use:

    docker stop fuseki

To restart a named container (it will remember the volume and port config)

    docker restart fuseki

## security

This docker uses shiro file to control access.  The shiro.ini file is pulled in from the /var/fuseki_home volume if the volume exists.  The default shiro.ini file is used from /tmp if the volume does not exist or has a permission problem. See [ini syntax](https://shiro.apache.org/configuration.html#ini-sections)

## Upgrading Fuseki

If you want to upgrade the Fuseki container named `fuseki` which use the data
volume `fuseki-data` as recommended above, do:

    docker pull dskow/jena-fuseki
    docker stop fuseki
    docker rm fuseki
    docker run -d --name fuseki --volumes-from fuseki-data dskow/jena-fuseki


## Data loading

Fuseki allows uploading of RDF datasets through the web interface and web
services, but for large datasets it is more efficient to load them directly
using the command line.

This docker image includes a shell script `load.sh` that invokes the
[tdbloader](https://jena.apache.org/documentation/tdb/commands.html)
command line tool and load datasets from the docker volume `/var/fuseki_home/staging`.


For help, try:

    docker run dskow/jena-fuseki ./load.sh

You will most likely want to load from a folder on the host computer by using
`-v`, and into a data volume that you can then use with the regular fuseki.

Before data loading, you must either stop the Fuseki container, or
load the data into a brand new dataset that Fuseki doesn't know about yet.
To stop the docker container you named `fuseki`:

    docker stop fuseki

The example below assume you want to populate the Fuseki dataset 'chembl19'
from the Docker data volume `fuseki-data` (see above) by loading the two files
`cco.ttl.gz` and `void.ttl.gz` from `/var/fuseki_home/staging` on the volume data
container:

    docker run --volumes-from fuseki-data \
       dskow/jena-fuseki \
	   ./load.sh chembl19 cco.ttl.gz void.ttl.gz

**Tip:** You might find it benefitial to run data loading from the data staging
directory in order to use tab-completion etc. without exposing the path on the
host. The `./load.sh` will expand patterns like `*.ttl` - you might have to
use single quotes (e.g. `'*.ttl'`) on the host to avoid them being expanded
locally.

If you don't specify any filenames to `load.sh`, all filenames directly under
`/var/fuseki_home/staging` that match these GLOB patterns will be loaded:

    *.rdf *.rdf.gz *.ttl *.ttl.gz *.owl *.owl.gz *.nt *.nt.gz *.nquads *.nquads.gz

`load.sh` populates the default graph. To populate named
graphs, see the `tdbloader` section below.

**NOTE**: If you load data into a brand new `/var/fuseki_home` volume, a new random
admin password will be set before you have started Fuseki.
You can either check the output of the data loading, or later override the
password using `-e ADMIN_PASSWORD=pw123`.


## Recognizing the dataset in Fuseki

If you loaded into an existing dataset, Fuseki should find the data after
(re)starting with the same data volume (see [Data
persistence](#Data_persistence) above):

    docker restart fuseki

If you created a brand new dataset, then in Fuseki go to *Manage datasets*,
click **Add new dataset**, tick **Persistent** and provide the database name
exactly as provided to `load.sh`, e.g. `chembl19`.

Now go to *Dataset*, select from the dropdown menu, and try out *Info* and *Query*.

**Tip**: It is possible to load a new dataset into the volume of a
running Fuseki server, as long as you don't "create" it in Fuseki before
`load.sh` has finished.


## Loading with tdbloader

If you have more advanced requirements, like loading multiple datasets or named graphs, you can
use [tdbloader](https://jena.apache.org/documentation/tdb/commands.html) directly together with
a [TDB assembler file](https://jena.apache.org/documentation/tdb/assembler.html).

Note that Fuseki TDB datasets are sub-folders in `/var/fuseki_home/databases/`.

You will need to provide the assembler file on a mounted Docker volume together with the
data:

    docker run --volumes-from fuseki-data \
	   dskow/jena-fuseki \
       ./tdbloader --desc=/var/fuseki_home/staging/tdb.ttl

## Customizing Fuseki configuration

If you need to modify Fuseki's configuration further, you can use the equivalent of:

    docker run --volumes-from fuseki-data -it ubuntu bash

and inspect `/var/fuseki_home` with the shell. Remember to restart fuseki afterwards:

    docker restart fuseki
