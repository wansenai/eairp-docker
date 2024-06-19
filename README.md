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
-	Several containers are used with Docker Compose: one for the mysql one for Redis and another for Eairp + Nginx. This allows the ability to run them on different machines for example.

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
