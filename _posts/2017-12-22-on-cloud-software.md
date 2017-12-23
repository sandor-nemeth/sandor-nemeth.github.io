---
published: false
title: On Cloud software
layout: post
---

In the last weeks I had several talks with collagues from both my old and my
new workplace on using microservices and - especially - using them in a
cloud-like environment (e.g in our context [Spring Cloud]). I think it makes
sense to summarize these thoughts up, so here they are.

## What problem do they solve?

Microservices, [Spring Cloud], service discovery, they all solve a very
specific set of problems, which can be roughly summarized up in 2 words:

> Elastic scaling

What does `Elastic scaling` mean? It means that:

> At any given point in time, a single service can be deployed onto any number
> of nodes, and this deployment can be changed at any point in time.

To explain this, let's see an example: imagine a webshop called
`my-super-webshop.com`. The following graph illustrates the rough traffic
during a normal workday:

![Daily traffic of my-super-webshop.com](/assets/images/daily_traffic.png)

_Note: the visitor number are in thousands._

This site has a quite varying traffic, and on special occasions (like christmas
or Black Friday) their traffic multiplies. This means that they have 2 options:

1. either they can plan for the traffic they would have on a Black Friday (which
  would mean keeping up lots of servers)
1. or they can create new instances based on the history they have and a simple
  traffic prediction algorithm

Obviously the 2nd choice is much more cost-effective. So basically their
solution is to host the site at a cloud provider (like Amazon), and based on
their prediction, provision new processing nodes and scale out their operations
as needed. But this also means, that basically at 4pm they can have 10
instances of their product details service (which delivers the product data for
display to the customer), and at 8pm they'd have 30 instances. In this kind of
variance, there is no way any Ops department would be able to manually deploy
and configure 20 additional instances (which they'd have to tear down in 2-3
hours anyways).

This is a very basic example of what is the problem which these highly
automated setups solve. There is no way anybody would be able to make this work
manually. 

[Spring Cloud]: http://projects.spring.io/spring-cloud/
