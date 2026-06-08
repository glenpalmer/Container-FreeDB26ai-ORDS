# Container-FreeDB26ai-ORDS

Information here is based on Docker / Podman installations of Oracle Free Database 26ai and ORDS 26.1.1 where Oracle Application Express (APEX) can be installed.  These details are based on the installation on Mac OS.

A version for Linux (Fedora) will follow shortly as there are subtle differences.

## Pull Containers

First pull the containers from the Oracle repository, this will be the latest version of the Database (26ai) and ORDS (26.1.1)

**Docker**
```
docker pull container-registry.oracle.com/database/free:latest
docker pull container-registry.oracle.com/database/ords:latest
```

**Podman**
```
podman pull container-registry.oracle.com/database/free:latest
podman pull container-registry.oracle.com/database/ords:latest
```

## Create Network

Create a network for the containers to be able to communicate

**Docker**
```
docker network create oracle-network
```

**Podman**
```
podman network create oracle-network
```

## Run Container DB

Run the container for the Database, first create a local directory structure to map a volume to so data can be persistent, else the data files from the Database will be removed when the container is stopped.

In this example the local directory ~/Podman/orafree25ai/oradata will be mapped to the conatiner directory /opt/oracle/oradata

Not the first time this container runs, it may take some time to complete as the database needs to be created.

**Docker**
```
docker run -d --name orafree26ai --hostname orafree26ai --network=oracle-network -p 1521:1521 -v ~/Container/orafree26ai/oradata:/opt/oracle/oradata container-registry.oracle.com/database/free:latest
```

**Podman**
```
podman run -d --name orafree26ai --hostname orafree26ai --network=oracle-network -p 1521:1521 -v ~/Podman/orafree26ai/oradata:/opt/oracle/oradata container-registry.oracle.com/database/free:latest
```

## Change Password

Run a local script on the Database container which will be used to set the password for the default SYS account.

**Docker**
```
docker exec orafree26ai ./setPassword.sh password
```

**Podman**
```
podman exec orafree26ai ./setPassword.sh password
```

## Install APEX

Either using Docker Desktop or Podman Desktop open the terminal, or from the local host machine use the following command to connect to bash on the container where APEX will be installed.

**Docker**
```
docker exec -u oracle -it orafree26ai /bin/bash
```

**Podman**
```
podman exec -u oracle -it orafree26ai /bin/bash
```

Once connected to the container, change directory to tmp and pull down the latest version of APEX.
```
cd /tmp

curl -O https://download.oracle.com/otn_software/apex/apex-latest.zip
```

Unzip the contents of the downloaded APEX zip.
```
unzip apex-latest.zip
```

Change directory to the `apex` directory.
```
cd apex
```

Connect to the database using SYS user and as SYSDBA, each section exit and reconnect as SYS.
```
sqlplus sys/password@localhost:1521/FREEPDB1 as sysdba
```

Once connected, create a new database file to install APEX.
```
create tablespace APEX datafile 'apex_001.dbf' size 300m autoextend on;

exit
```

Run the installation of apex, there are 2 commands where one will us local image directory and the other using the Oracle CDN which serves the images.
```
sqlplus sys/password@localhost:1521/FREEPDB1 as sysdba

@apexins.sql APEX APEX TEMP /i/
@apexins.sql APEX APEX TEMP https://static.oracle.com/cdn/apex/26.1.0/

exit
```

Unlock the APEX Public User account.
```
sqlplus sys/password@localhost:1521/FREEPDB1 as sysdba

alter user apex_public_user account unlock;

exit
```

Set the password for the ADMIN user of APEX.
```
sqlplus sys/password@localhost:1521/FREEPDB1 as sysdba

@apxchpwd.sql

exit
```

Install the REST services configuration.
```
sqlplus sys/password@localhost:1521/FREEPDB1 as sysdba

@apex_rest_config.sql

exit
```

Exit from the terminal session connected to the Database container.

**Docker**
```
exit
```

**Podman***
```
exit
```

## Run Container ORDS

Run the container for the Oracle RESTful Data Services (ORDS), first create a local directory structure to map a volume to so data can be persistent, else the data files from the ORDS configuration will be removed when the container is stopped.

In this example the local directory ~/Podman/ords2611/config will be mapped to the conatiner directory /etc/ords/config

Notice the first time this container is run, the 'install' option is used.

**Docker**
```
docker run --rm -it --name ords2611 --network oracle-network -p 8080:8080 -e DBHOST=orafree26ai -e DBPORT=1521 -e DBSERVICE=freepdb1 -e ORACLE_PWD=passsword -v ~/Container/oraords2611/config:/etc/ords/config container-registry.oracle.com/database/ords:latest install
```

**Podman**
```
podman run --rm -it --name ords2611 --network oracle-network -p 8080:8080 -e DBHOST=orafree26ai -e DBPORT=1521 -e DBSERVICE=freepdb1 -e ORACLE_PWD=passsword -v ~/Podman/ords2611/config:/etc/ords/config container-registry.oracle.com/database/ords:latest install
```

The container will run in the terminal and as the 'install' option has been used, prompts will be returned to configure ORDS which will be saved in the config file.  Note that the contaier of the database (orafree26ai) must be running as ORDS will be installed.  The following prompts will appear:

```
INFO : Running Oracle REST Data Services CLI command.
2026-06-06T13:24:22Z INFO   ORDS has not detected the option '--config' and this will be set up to the default directory.

ORDS: Release 26.1 Production on Sat Jun 06 13:24:24 2026

Copyright (c) 2010, 2026, Oracle.

Configuration:
  /etc/ords/config

The configuration folder /etc/ords/config does not contain any configuration files.

Oracle REST Data Services - Interactive Install

  Enter a number to select the database connection type to use
    [1] Basic (host name, port, service name)
    [2] TNS (TNS alias, TNS directory)
    [3] Custom database URL
  Choose [1]: 1
  Enter the database host name [localhost]: orafree26ai
  Enter the database listen port [1521]: 1521
  Enter the database service name [orcl]: freepdb1
  Provide database user name with administrator privileges.
    Enter the administrator username: sys
  Enter the database password for SYS AS SYSDBA:

Retrieving information.
ORDS is not installed in the database. ORDS installation is required.

  Enter a number to update the value or select option A to Accept and Continue
    [1] Connection Type: Basic
    [2] Basic Connection: HOST=orafree26ai PORT=1521 SERVICE_NAME=freepdb1
           Administrator User: SYS AS SYSDBA
    [3] Database password for ORDS runtime user (ORDS_PUBLIC_USER): <generate>
    [4] ORDS runtime user and schema tablespaces:  Default: SYSAUX Temporary TEMP
    [5] Additional Feature: Database Actions
    [6] Configure and start ORDS in Standalone Mode: Yes
    [7]    Protocol: HTTP
    [8]       HTTP Port: 8080
    [9]   APEX static resources location: null
    [A] Accept and Continue - Create configuration and Install ORDS in the database
    [Q] Quit - Do not proceed. No changes
  Choose [A]: A
```

Stop the container for ORDS once the install is complete.  Notice that once the container has been stopped, it will be removed as the '--rm' tag was used on the installation.  The next time the container is run it'll use the 'serve' option instead of 'install' and the config file created on the install will be used.

**IMPORTANT!!!**

When using images from a static location and not being served by the CDN (for example developing on a laptop with no internet connection), a quick update to the 'settings.xml' for standalone install should include a pointer to a location where the images can be served from.  In this following example, the images folder from APEX download should be copied to this location so that ORDS can serve those images.

```
<entry key="standalone.static.path">/etc/ords/config/global/static/images</entry>
```

This entry should appear in the following location which is also mapped as a bind volume : -

`etc/ords/config/global/settings.xml`

## Restart the ORDS container

From the local machine terminal, run the ORDS container again using the 'serve' option.
**Docker**
```
docker run -it --name ords2611 --network oracle-network -p 8080:8080 -v ~/Container/ords2611/config:/etc/ords/config container-registry.oracle.com/database/ords:latest serve
```

**Podman**
```
podman run -it --name ords2611 --network oracle-network -p 8080:8080 -v ~/Podman/ords2611/config:/etc/ords/config container-registry.oracle.com/database/ords:latest serve
```

## Open APEX

If using the static CDN to serve the images which was an option during the APEX installation, navigate to the following to complete the installation:-

```
https://localhost:8080/ords](http://localhost:8080/ords/_/landing
```

First connect to the following: -

```
Workspace : INTERNAL
Username  : ADMIN
Password  : (the password used in the change password script during installation)
```
