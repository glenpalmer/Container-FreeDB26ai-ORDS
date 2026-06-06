# Container-FreeDB26ai-ORDS

Information here is based on Docker / Podman installations of Oracle Free Database 26ai and ORDS 26.1.1 where Oracle Application Express (APEX) can be installed.  These details are based on the installation on Mac OS.

A version for Linux (Fedora) will follow shortly as there are subtle differences.

## Pull Containers

First pull the containers from the Oracle repository, this will be the latest version of the Database (26ai) and ORDS (26.1.1)

Docker
```
docker pull container-registry.oracle.com/database/free:latest
docker pull container-registry.oracle.com/database/ords:latest
```

Podman
```
podman pull container-registry.oracle.com/database/free:latest
podman pull container-registry.oracle.com/database/ords:latest
```

## Create Network

Create a network for the containers to be able to communicate

Docker
```
docker network create oracle-network
```

Podman
```
podman network create oracle-network
```

## Run Container DB

Run the container for the Database, first create a local directory structure to map a volume to so data can be persistent, else the data files from the Database will be removed when the container is stopped.

In this example the local directory ~/Podman/orafree25ai/oradata will be mapped to the conatiner directory /opt/oracle/oradata

Not the first time this container runs, it may take some time to complete as the database needs to be created.

Docker
```
docker run -d --name orafree26ai --hostname orafree26ai --network=oracle-network -p 1521:1521 -v ~/Container/orafree26ai/oradata:/opt/oracle/oradata container-registry.oracle.com/database/free:latest
```

Podman
```
podman run -d --name orafree26ai --hostname orafree26ai --network=oracle-network -p 1521:1521 -v ~/Podman/orafree26ai/oradata:/opt/oracle/oradata container-registry.oracle.com/database/free:latest
```

## Change Password

Run a local script on the Database container which will be used to set the password for the default SYS account.

Docker
```
docker exec orafree26ai ./setPassword.sh password
```

Podman
```
podman exec orafree26ai ./setPassword.sh password
```

## Install APEX

Either using Docker Desktop or Podman Desktop open the terminal, or from the local host machine use the following command to connect to bash on the container where APEX will be installed.

Docker
```
docker exec -u oracle -it orafree26ai /bin/bash
```

Podman
```
podman exec -u oracle -it orafree26ai /bin/bash
```

Once connected to the container, change directory to tmp and pull down the latest version of APEX.

```
cd /tmp

curl -O https://download.oracle.com/otn_software/apex/apex-latest.zip
```
