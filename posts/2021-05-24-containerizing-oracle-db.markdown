---
title: Containerizing Oracle Database
---

## Intro

In this post, we will walk-through on how to make a **Docker** container of an **Oracle Database**.

The end result will be a running **Oracle DB Instance** that has an application schema set up and 
can persist our application data through restarts using a **Volume**

The steps are based on the instructions for containerizing a **Single Instance** of the 
**Oracle DB** presented at:
https://github.com/oracle/docker-images/blob/main/OracleDatabase/SingleInstance/README.md

---

## Versions

* Docker Engine: 20.10.6
* Docker Compose: 1.29.1
* Oracle Database: 12c Release 2 (12.2.0.1) Enterprise Edition and Standard Edition 1

---

## Oracle dockerfiles

First things first, we have to clone **Oracle's** repository that contains useful files for the generation
of the **Docker Image** of the **Oracle Database**. The repository can be found at:
https://github.com/oracle/docker-images

We are interested in the **Single Instance** case, so the files that we focus on is in the path 
`OracleDatabase/SingleInstance` 

---

## Oracle Database Binaries

In order to create the **Docker Image** we have to have the **Oracle Database Binaries** for the version we
want to build.

These can be downloaded by **Oracle Technology Network**.

Once downloaded, the **unzipped** archive **linuxx64_12201_database.zip** should be:
placed inside

```shell
<path_to_oracle_docker_images_cloned_repo>/OracleDatabase/SingleInstance/dockerfiles/12.2.0.1/
```

---

## Docker Image Creation

After the above step, with the **Docker daemon** running, we can build the **Docker Image** of 
the database issuing a command similar to the below one _(Note that the current working directory
should be something similar to `<path_to_oracle_docker_images_cloned_repo>/OracleDatabase/SingleInstance/dockerfiles`)_:

```shell
./buildDockerImage.sh -v 12.2.0.1 -e
```

This command will generate a **oracle/database:12.2.0.1-ee** image in the **Local Docker Image Registry**, and then, we 
could build containers based on that image.





