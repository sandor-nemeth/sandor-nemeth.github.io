---
published: false
date: 2017-12-26
title: Running Spring Cloud on Swarm
layout: post
---

_Note: I'll update this post in installments, and at the end I'll create a TOC
also. But as this'll be a lather large post, I'll keep updating this post
until it is finished._

Uh, I need to apologise, I wasn't really thinking that I won't have that much
time inbetween the last post and now. However as Christmas is around I have
some time to summarize my thoughts on how to build a [Swarm] cluster, and then
put a (mostly) complete Spring Cloud environment on it - supporting
Swarm features like `docker service scale`.

In this post the target is to set up a fully operational scalable environment,
and start deploying microservices. The applications will be Java apps created
with [Spring Boot] and [Spring Cloud], and the environment will be mostly the
Netflix cloud environment utilizing [Eureka], [Zuul], [Hystrix] and [Ribbon].
Planned follow-up posts will explain Hystrix and Ribbon, right now the focus
is on having an environment up and running, and then scaling it by adding
more nodes to the cluster and scaling the services out.


- Service Discovery: Spring Cloud Eureka 
- Central Configuration: Spring Cloud Config Server
- Trace: Spring Cloud Zipkin
- API Gateway: Spring Cloud Zuul
- And a few applications for demonstration purposes.

## Service Discovery - Eureka



## Centralized configuration management - Spring Cloud Config server

## Tracing - Zipkin

## API Gateway - Zuul

## Demo applications


[Swarm]: https://docs.docker.com/engine/swarm/
[Spring Boot]: https://projects.spring.io/spring-boot/
[Spring Cloud]: http://projects.spring.io/spring-cloud/
[Eureka]: https://github.com/Netflix/eureka
[Zuul]: https://github.com/Netflix/zuul
[Hystrix]: https://github.com/Netflix/hystrix
[Ribbon]: https://github.com/Netflix/ribbon
