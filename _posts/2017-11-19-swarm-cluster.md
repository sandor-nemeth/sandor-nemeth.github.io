---
published: false
title: Creating a Swarm cluster
layout: post
---
Today I am starting a series of articles on how to build an on-premise cluster. Admittedly, this is a demo-setup, which means that one should be precisely aware what needs to happen to make this a setup ready for production. 

OK, with the disclaimer noted, let's get going on setting up a Docker Swarm cluster. 

First let's create 3 VMs which will be the base of our cluster: 

```bash
docker-machine create --driver virtualbox swarm-1
docker-machine create --driver virtualbox swarm-2
docker-machine create --driver virtualbox swarm-3
```

This will produce 3 virtual machines running Docker. The status of the machines can be checked with:

```bash
docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
swarm-1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.10.0-ce
swarm-2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.10.0-ce
swarm-3   -        virtualbox   Running   tcp://192.168.99.102:2376           v17.10.0-ce
```

If you see this, then we are good to go for the next step!


