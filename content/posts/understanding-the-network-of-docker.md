+++ 
draft = true
date = 2022-10-15T06:41:21-03:00
title = ""
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

# Understanding the network of the Docker

## Introduction

This article aims to explain in detail, and with examples, how the data communication in network of Docker works. We will talk about the defaults available options and how to create new networks.

This text has a target audience people that already know a little about Docker, but not they hadn't mastered the Docker networks yet. In case you think the text confused or missing some important information, please, report it in a comment.

What Docker calls network, in fact is a abstraction, created for facilitate the management of data communication between containers and external nodes of docker environment.

Don't confuse the Docker network with the already known network, used to group the IP addresses (ex 192.168.10.0/24), that way, ever I need mention this second network type, I will mention as "IP network".

## Defaults Docker networks

The Docker is made available with three defaults networks. Those networks offer specifics settings for data traffic management. To view those interfaces, just use the following command:

```
    docker network ls
```

The return will be:

```
    NETWORK ID          ID              DRIVER
    ab09673e9b98        bridge          bridge
    763f9ed88176        none            null
    24a960a6f20         host            host
```

## Bridge

Each container started on docker is associated to a specific network, and this is the default network for any container, that was not been explicitly specified.

It provides containers a interface that make a bridge with the docker0 interface of the Docker host. This interface will automatically receive the next available address in IP network 172.17.0.0/16.

All containers that is in this network can communicate through the TCP/IP protocol, in other words, if you know what is the container IP address that you want to connect, is possible send traffic to it, since all containers is in the same IP network (172.17.0.0/16).

A detail to observe is how the IPs is automatically assigned, it's not a trivial task to discover what is the container destination IP. For helps with this localization, the docker offers in container initialization a "-link" option.

> Is a important point that "-link" is a already lagged option, and is advised not to use it. I just will explain this feature to gain a better understanding of the legacy.

The option "-link" is responsible for assign the intended container IP to its name, in other words, if you starts a container from a MySQL Docker image with "db" name, and following starts other with "app" name, from a "tutum/apache-php" docker image, wishing the last container can to connect with MySQL using the name "db" of container, is just start with the following way, for both containers:

```
    docker run -d --name db -e MYSQL_ROOT_PASSWORD=mypassword mysql

    docker run -d -p 80:80 --name app --link db tutum/apache-php
```

After to execute the commands, the container that has the "app" name, could to connect the MySQL container using the name "db", in other words, every time it to try access the "db" name its will automatically resolved as network IP 172.17.0.0/16, that the MySQL container got in it initialization.

For testing, we will use a exec feature to run a command inside a already existing container. For it we will use the container name as parameter for the following command:

```
    docker exec -ti app ping db
```

The above command is responsible for executing the "ping db" command" inside the "app" container, in other words, the "app" container will send icmp packages, that is used for connection testing between two hosts, for "db" address. This name "db" is translated for the IP that's started container from MySQL image got from starts.

**Exemple:** The "db" container started first, and got the 172.17.0.2 IP. The container "app" started following it and got the 172.17.0.1 IP. When the "app" container run the "ping db" command, indeed it will send icmp packages for 172.17.0.2 address.

> Attention: The "-link" option may be confusing, since it does not create a IP network link between containers, since the communication between it already is possible, even without the link option is configured. How was informed in previous paragraph, it just facilitates the names translation for the dynamic IP got from initialization.

The configured containers for this network will have the external traffic possibility, using the network IP routes sign on docker host, in other words, if the docker host has internet access, the containers in question will have too.

In this network is possible to expose container ports for every assets that can access the docker host.

## None

This network aims to to isolate the container for external communications, in other words, it will not be able to receive none interface for external communication. The sole network IP interface will be the localhost.

This network usually is used for containers that handle just files, without 