# Summary

The Beacon v2 Reference Implementation is a software that must be installed **locally** in a Linux server/workstation.

The software is distributed in **two different GitHub repositories**: 

1. The data ingestion tools ([beacon2-ri-tools](https://github.com/mrueda/beacon2-ri-tools)).

2. The API ([beacon2-ri-api](https://github.com/EGA-archive/beacon2-ri-api)). 

!!! Note "Tip"
    The data ingestion tools can function without the API, so you can download one without the other.

The software also needs external dependencies to work. In this regard, we provide several alternatives for download and installation.


Before starting, it will be necessary to install:

* [Docker + Docker-compose](https://docs.docker.com/engine/install/ubuntu/)

## Containerized (Recommended)

### Method 1: All containers in one

##### Setting up all the containers, `beacon2-ri-tools` and `beacon2-ri-api` together using the [beacon2-ri-api](https://github.com/EGA-archive/beacon2-ri-api) repository.

Clone `beacon2-ri-api` GitHub repository.

    git clone https://github.com/EGA-archive/beacon2-ri-api.git

From now on, the commands should be executed from the deploy directory.

    cd beacon2-ri-api/deploy

Light up all the containers.

    docker network create my-app-network
    docker-compose up -d --build

##### Deploying external tools in `beacon2-ri-tools` container

!!! Danger "External Tools Architecture Alert"

    Please note that the external tools are compiled specifically for `x86-64` architecture. Consequently, they are not compatible with newer ARM-based Macs.

This step will inject the external tools and DBs into the image and modify the configuration files. It will also run a test to check that the installation was successful. Note that running `deploy_external_tools.sh` will take some time (and disk space!!!).

Go inside `beacon2-ri-tools` container.

    docker exec -it deploy_beacon-ri-tools_1 bash
    bash /usr/share/beacon-ri/beacon2-ri-tools/BEACON/bin/deploy_external_tools.sh

!!! Important
    The container's name should be similar to `deply_beacon-ri-tools_1`. It can be extracted from docker ps (`docker exec–it<container_name>bash`)

With `mongo-express` it is possible to see the contents of the data base at http://localhost:8081.

For more installation options, please follow the instructions provided in the [README](https://github.com/EGA-archive/beacon2-ri-api/blob/master/deploy/README.md).

### Method 2: Independent containers

Download the `beacon2-ri-tools` container and the `beacon2-ri-api` container independently.

#### Data ingestion tools (`beacon2-ri-tools`)

!!! Note

    The original version of the tools is now deprecated and unmaintained. Due to restricted access to the original EGA-archive repository, development has continued in a new forked repository. The latest and actively maintained version of `beacon2-ri-tools` can be found in the [following repository](https://github.com/mrueda/beacon2-ri-tools).

See installation instructions [here](https://github.com/mrueda/beacon2-ri-tools/blob/main/docker/README.md).

!!! Danger "External Tools Architecture Alert"

    Please note that the external tools are compiled specifically for `x86-64` architecture. Consequently, they are not compatible with newer ARM-based Macs.

!!! Warning "About MongoDB Community version"
    We're deliberately installing `mongodb` package provided by Ubuntu, which is **not** maintained by MongoDB Inc. The _official_ distribution of MongoDB Community is called `mongodb-org` and can be obtained [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu).

!!! Important
    We will be installing **MongoDB** with a _single node/server_ configuration. Note that MongoDB allows for other topologies such as as _replica sets_ that we won't be covering in this documentation.

#### Beacon v2 REST API (`beacon2-ri-api`)

See installation intructions [here](https://github.com/EGA-archive/beacon2-ri-api/blob/master/deploy/README.md)

## Non containerized (`beacon2-ri-tools`)

!!! Warning "Message"
    This alternative requires a few more steps than the containerized one, but it gives the deployer more control over the tools. Only recommended for **advanced** users.

See instructions [here](https://github.com/mrueda/beacon2-ri-tools/blob/main/non-containerized/README.md). 
