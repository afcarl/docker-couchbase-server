# Couchbase Server on Docker Containers

This is a Dockerfile and supporting scripts for running
[Couchbase Server](http://couchbase.com/) in a
[Docker](http://www.docker.com/) container.

Originally a personal research project, this project has been superseded
by official Docker related resources described below.

**NOTE** You might want to consider these alternative projects:

* [Couchbase Docker image on dockerhub](https://hub.docker.com/u/couchbase/server)
* [Dockerfiles and configuration scripts for the Docker Hub Official Couchbase images](https://github.com/couchbase/docker)
* [Run Couchbase Server under Docker + Kubernetes](https://github.com/tleyden/couchbase-kubernetes)
* [Run Couchbase Server under Docker + CoreOS](https://github.com/couchbaselabs/couchbase-server-coreos)

## Prepare Docker Host

Some preparation of the host operating system running the Docker daemon is
required prior to launching Docker containers. The exact preparation steps
differ depending on the OS distribution.

### Debian, Ubuntu, CentOS or RHEL

**Note about open file limits and locked memory**: You'll need to increase
the number of open files and locked memory available to Couchbase Server
containers on the Docker host.

To do so, first access a shell on the host machine and create the docker
daemon initialization file, `/etc/init/docker.conf`:

```
sudo $EDITOR /etc/init/docker.conf
```

Then add the following lines to the file:

```
limit memlock unlimited unlimited
limit nofile 262144 262144
```

You'll need to restart the Docker daemon after making the above changes. These changes will affect the Docker daemon and all of its child processes,
including containers.

### CoreOS

This project will not be supporting CoreOS in its current form.

## Run a Couchbase Server Container

Now, you can run a Couchbase Server Docker container.

If you have not already, clone this project repository to your Docker host:

```
git clone https://github.com/brianshumate/docker-couchbase-server.git
```

Then, use the following commands to run a container based on this project:

```
cd docker-couchbase-server
INT=`ip route | awk '/^default/ { print $5 }'`
ADDR=`ip route | egrep "^[0-9].*$INT" | awk '{ print $9 }'`
exec sudo docker run -i -d -t -e DOCKER_EXT_ADDR=$ADDR \
-e "SERVICE_NAME=couchbase-server" -e "SERVICE_TAGS=couchbase" \
-v /home/core/data/couchbase:/opt/couchbase/var \
-p 11210:11210 -p 8091:7081 -p 8092:8092 \
jbs_cb:dockerfile
```

If your Docker host is running CoreOS, use the included `coreos.script`:

```
cd docker-couchbase-server
exec sudo ./bin/coreos.script
```

You can also get a 3 node cluster going with the included
`multi-node-cluster` script:

```
cd docker-couchbase-server
exec sudo ./bin/multi-node-cluster
```

## macOS Instructions

Docker is nativelt supported on macOS. To learn more about official Docker
macOS support, consult the 
[Docker for Mac documentation](https://docs.docker.com/docker-for-mac/).

Build the `Dockerfile`:

```
git clone https://github.com/brianshumate/docker-couchbase-server
cd docker-couchbase-server
docker build -t "brianshumate_cb:dockerfile" .
```

Run it (optionally with `SERVICE_NAME` and `SERVICE_TAGS` for Consul:

```
docker run -i -d -t -v $HOME/tmp/couchbase:/opt/couchbase/var \
-e "SERVICE_NAME=couchbase-server" -e "SERVICE_TAGS=couchbase" \
-p 11210:11210 -p 8091:7081 -p 8092:8092 \
--rm brianshumate_cb:dockerfile
```

## Resources

The following are some additional handy resources related to operating
Couchbase Server in a Docker environment:

* [Run Couchbase Server under Docker + CoreOS](https://github.com/couchbaselabs/couchbase-server-docker)
* [Dockerfiles and configuration scripts for the Docker Hub Official Couchbase images](https://github.com/couchbase/docker)
* [Running Couchbase Cluster Under CoreOS on AWS](http://tleyden.github.io/blog/2014/11/01/running-couchbase-cluster-under-coreos-on-aws/)
* [How I built couchbase 2.2 for docker](https://gist.github.com/dustin/6605182)
* [Couchbase Server / Docker Index](https://index.docker.io/u/dustin/couchbase/)
* [Running Couchbase Cluster Under Docker](http://tleyden.github.io/blog/2013/11/14/running-couchbase-cluster-under-docker/)
* [Couchbase Docker container](https://github.com/ncolomer/docker-templates/tree/master/couchbase)
