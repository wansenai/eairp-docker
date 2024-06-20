# What is Eairp

[Eairp](https://github.com/wansenai/eairp) (Enterprise AI Resource Planning) is a comprehensive resource planning system for enterprises, aimed at optimizing and integrating various operational processes, improving management efficiency, and reducing operating costs. 
EAIRP includes multiple functions such as material procurement, financial budgeting, inventory management, billing management, and user role organization management. 
It also introduces advanced AI assistants to provide intelligent management solutions for enterprises.

# Table of contents
- [Introduction](#introduction)
- [How to use this image](#how-to-use-this-image)
    -	[Pulling existing image](#pulling-an-existing-image)
        -	[Using docker run](#using-docker-run)
        -	[Using docker-compose](#using-docker-compose)
    -	[Building](#building)
- [Upgrading Eairp](#upgrading-eairp)
- [Troubleshooting](#troubleshooting)
- [Details for the eairp image](#details-for-the-eairp-image)
    -	[Configuration Options](#configuration-options)
    -	[Passing JVM options](#passing-jvm-options)
    -	[Remote Debugging](#remote-debugging)
    -	[Configuration Files](#configuration-files)
    -	[Miscellaneous](#miscellaneous)
- [For Maintainers](#for-maintainers)
    -	[Update Docker Images](#update-docker-images)
    -	[Testing Docker Images](#testing-docker-images)
    -	[Clean Up](#clean-up)
- [License](#license)
- [Support](#support)
- [Contribute](#contribute)
- [Credits](#credits)

# Introduction

The goal is to provide a production-ready Eairp system running in Docker. This is why:

-	The OS is based on Debian and not on some smaller-footprint distribution like Alpine
-	Several containers are used with Docker Compose: one for the mysql one for Redis and another for Eairp + Nginx. This allows the ability to run them on different machines.

# How to use this image

You should first install [Docker](https://www.docker.com/) on your machine.

Then there are several options:

1.	Pull the eairp image from DockerHub.
2.	Get the [sources of this project](https://github.com/wansenai/eairp) and build them.

### Using docker run

Start by creating a dedicated docker network:

```console
docker network create -d bridge eairp-nw
```

Then run a container for your MySQL database and make sure it is configured to use UTF8 encoding.

#### Starting MySQL

We will bind mount two local directories to be used by the MySQL container:
-	one to be used at database initialization to set permissions (see below), 
-	another to contain the data put by Eairp inside the MySQL database, so that when you stop and restart MySQL you don't find yourself without any data.

For example:
-	`/usr/local/mysql-init`
-	`/usr/local/mysql`

You need to make sure these directories exist, and then copy the `eairp.sql` file to the `/usr/local/mysql-init` directory. (you can name it the way you want, for example `init.sql`).

Note: Make sure the directories you are mounting into the container are fully-qualified, and aren't relative paths.

```console
docker run --net=eairp-nw \
           -d \
           --name mysql-eairp \
           -p 3306:3306 \
           -v /usr/local/mysql:/var/lib/mysql \
           -v /usr/local/mysql-init:/docker-entrypoint-initdb.d \
           -e MYSQL_ROOT_PASSWORD=123456 \
           -e MYSQL_USER=eairp \
           -e MYSQL_PASSWORD=123456 \
           -e MYSQL_DATABASE=eairp \
           mysql:8.3 \
           --character-set-server=utf8mb4 \
           --collation-server=utf8mb4_bin \
           --explicit-defaults-for-timestamp=1
```
OR
```console
docker run --net=eairp-nw -d --name mysql-eairp -p 3306:3306 -v /usr/local/mysql:/var/lib/mysql -v /usr/local/mysql-init:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_USER=eairp -e MYSQL_PASSWORD=123456 -e MYSQL_DATABASE=eairp -d mysql:8.3 --character-set-server=utf8mb4 --collation-server=utf8mb4_bin --explicit-defaults-for-timestamp=1
```

You should adapt the command line to use the passwords that you wish for the MySQL root password.

#### Starting Redis

We will bind mount a local directory to be used by the Redis container to contain the data put by Eairp inside the database, so that when you stop and restart Eairp you don't find yourself without any data. For example:

-	`/usr/local/redis/data`
-	`/usr/local/redis/redis.conf`

You need to make sure this directory exists, before proceeding.

Note Make sure the directory you specify is specified with the fully-qualified path, not a relative path.

```console
docker run --net=eairp-nw \
           -d \
           --name redis-eairp \
           -p 6379:6379 \
           -v /usr/local/redis/redis.conf:/etc/redis/redis.conf \
           -v /usr/local/redis/data:/data \
           -e SPRING_REDIS_PASSWORD=123456
           redis:latest
```
OR
```console
docker run --net=eairp-nw -d --name redis-eairp -v /usr/local/redis/redis.conf:/etc/redis/redis.conf -v /usr/local/redis/data:/data -p 6379:6379 redis:latest
```

You should adapt the command line to use the passwords that you wish for the Redis password.

#### Starting Eairp

We will also bind mount a local directory for the Eairp log permanent directory (contains application config and state), for example:

-	`/usr/local/eairp`

Note Make sure the directory you specify is specified with the fully-qualified path, not a relative path.

Ensure this directory exists, and then run XWiki in a container by issuing one of the following command.

```console
docker run --net=eairp-nw \
           -d \
           --name eairp \
           -p 8088:8088 \
           -p 3000:80 \
           -v /usr/local/eairp:/application/log \
           -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql-eairp:3306/eairp \
           -e SPRING_DATASOURCE_USERNAME=eairp \
           -e SPRING_DATASOURCE_PASSWORD=123456 \
           -e SPRING_REDIS_HOST=redis-eairp \
           -e SPRING_REDIS_PORT=6379 \
           -e SPRING_REDIS_PASSWORD=123456 \
           -e API_BASE_URL=http://localhost:8088/erp-api \
           wansenai/eairp:latest
```
OR
```console
docker run --net=eairp-nw -d --name eairp -p 8088:8088 -p 3000:80 -v /usr/local/eairp:/application/log -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql-eairp:3306/eairp -e SPRING_DATASOURCE_USERNAME=eairp -e SPRING_DATASOURCE_PASSWORD=123456 -e SPRING_REDIS_HOST=redis-eairp  -e SPRING_REDIS_PORT=6379 -e SPRING_REDIS_PASSWORD=123456 -e API_BASE_URL=https://eairp.cn/erp-api wansenai/eairp:latest
```

Note the Eairp uses Spring DataSource in Spring Boot as the data source to connect to the database

Please don’t forget to add the MySQL database connection environment variables (`SPRING_DATASOURCE_URL`) and the Redis database connection environment variables (`SPRING_REDIS_HOST`) with the names of the MySQL container and Redis container created earlier so that Eairp knows the location of its databases.

If you want to deploy on your server, please modify the value of the `API_BASE_URL` environment variable, for example:
-	`http://eairp.cn/erp-api`
-	`https://eairp.cn/erp-api`

### Using docker-compose

Another solution is to use the Docker Compose files we provide, You must download 5 files from [eairp-docker](https://github.com/wansenai/eairp-docker) repository, they are:

-	`.env`
-	`Dockerfile`
-	`docker-compose.yaml`
-	`start.sh`
-	`mysql-scripts/eairp.sql`

