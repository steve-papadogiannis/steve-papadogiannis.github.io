---
title: Containerizing Oracle Database
---

## Intro

In this post, we will walk-through on how to make a **Docker** container of an **Oracle Database**.

The end result will be a running **Oracle DB Instance** that has an application schema set up and 
can persist our application data through restarts using a **Volume**

The steps are based on the instructions for containerizing a **Single Instance** of the 
**Oracle DB** presented at: [README.md](https://github.com/oracle/docker-images/blob/main/OracleDatabase/SingleInstance/README.md)

---

## Versions

* Docker Engine: 20.10.6
* Docker Compose: 1.29.1
* Oracle Database: 12c Release 2 (12.2.0.1) Enterprise Edition

---

## Oracle dockerfiles

First things first, we have to clone **Oracle\'s** repository that contains useful files for the generation
of the **Docker Image** of the **Oracle Database**. The repository can be found at:
[https://github.com/oracle/docker-images](https://github.com/oracle/docker-images)

We are interested in the **Single Instance** case, so the files that we focus on is in the path 
`OracleDatabase/SingleInstance` 

---

## Oracle Database Binaries

In order to create the **Docker Image** we have to have the **Oracle Database Binaries** for the version we
want to build.

These can be downloaded by **Oracle Technology Network**.

Once downloaded, the **zipped** archive **linuxx64_12201_database.zip** should be
placed inside:

```shell
<path_to_oracle_docker_images_cloned_repo>/OracleDatabase/SingleInstance/dockerfiles/12.2.0.1/
```

---

## Docker Image Creation

After the above step, with the **Docker daemon** running, we can build the **Docker Image** of 
the database issuing a command similar to the below one _(Note that the current working directory
should be something similar to `<path_to_oracle_docker_images_cloned_repo>
/OracleDatabase/SingleInstance/dockerfiles`)_:

```shell
./buildDockerImage.sh -v 12.2.0.1 -e
```

This command will generate an **oracle/database:12.2.0.1-ee** image in the **Local Docker Image Registry**, and then, we 
could build containers based on that image.

---

## Docker Container Creation

In this step, we would like to setup a container which would have a running instance of the
**Oracle Database Version** we specified above and would persist the data across restarts.

On first startup of the container a new database will be created, and at that time
we can run the initialization scripts in order to have an application database ready.

For the scripts to be run at post-setup phase they should be included in 
`/opt/oracle/scripts/setup` directory inside the container

### Dockerfile

To extend the setup with the initialization of the application database we could create
a **Dockerfile** to add that step of copying the necessary files in the **post-setup** directory.

<script src="https://gist.github.com/steve-papadogiannis/98180985eb1086920769062dbaaa4ff5.js"></script>

Inside the local directory `scripts`, we have to have the initialization scripts, prefixed
with a number to preserve the order we want them to run.

### Docker Compose Configuration File

In order to avoid specifying a long, easy-to-forget command to bring up the container,
we can define a **Docker Compose** `.yaml` file to declare the container\'s properties:

<script src="https://gist.github.com/steve-papadogiannis/1c8df5960cc86508e6f0b2377e3b347c.js"></script>

When the `docker-compose` is invoked to bring up the container it will search in the same
folder for the **Dockerfile** we specified above, create the container\'s **volume**, named
**oracle-database**, expose all necessary ports and set an environment variable
with the desired **Oracle Password**. 

The volume will be bound with 
**/opt/oracle/oradata** directory in order to preserve the database data across restarts. 

The **8080** port is optional as it is used for a configuration interface 
(**Oracle Application Express**) that is
out of scope for the post. 

The specified **password** will be used for the **system** users 
(**SYS**, **SYSTEM** and **PDB_ADMIN**)

If the setup is successful the below message should be printed somewhere in the 
container\'s logs:

```shell
#########################
DATABASE IS READY TO USE!
#########################
```

---

## Initialization scripts

If the application database is already accessible, or we have access to a reference schema,
**Oracle Database** gives a nice util tool where a privileged user can export the schema,
with or without the data, to a set of ordered scripts. That helps to recreate the schema
from these scripts when run in the correct order:

![Fig. 1: Exported scripts](../images/exported_scripts.png)

The scripts from 2 to 10 are the exported ones.

The first script is one that we define in order to 
instruct some database management commands.

<script src="https://gist.github.com/steve-papadogiannis/002b54bbf82d2e413db488285a4ed014.js"></script>

Due to the fact that **12c** version of **Oracle Database** introduced a different architecture
on the **Database**, now you can have a **CDB** (container database) that contains several
**PDB**s (pluggable databases), so in order to have our application schema to a specific **PDB**
(PDB1) we append this statement in the start of every script: **ALTER
SESSION
SET CONTAINER = ORCLPDB1;**

The setup demonstrated in the script is arbitrary and can vary based on the needs of the
application at hand.

---

## Summary

The aforementioned steps should result to a running **Oracle Database Instance** that holds
the **Application Schema**.

It should be accessible with the below connection info:

<div class="format-inner-table">

| Property | Value            |
|----------|------------------|
| Host     | localhost        |
| Service  | ORCLPDB1         |
| User     | DEV_REF_12_2_0_1 |
| Password | 1234             |
| Port     | 1521             |

</div>






