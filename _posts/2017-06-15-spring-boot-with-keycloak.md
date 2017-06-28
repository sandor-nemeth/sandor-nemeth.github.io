---
layout: post
title: Secure a Spring Boot Rest app with Spring Security and Keycloak
categories: java keycloak authentication spring
---

Today I wanted to explore [Keycloak](http://www.keycloak.org/), and decided
to set up a very simple Spring Boot microservice which handles authentication
and authorization with Spring Security, using Keycloak as my authentication
source.

As it turns out, it is pretty easy to set this thing up, but there are a few
tricks which I want to describe as not totally obvious.

# Set up Keycloak

First we'll need a Keycloak instance so let's fire up the Docker container
provided by Jboss:

```bash
docker run -d \
  --name keycloak \
  -e KEYCLOAK_USER=admin \
  -e KEYCLOAK_PASSWORD=admin \
  -p 9001:8080 \
  jboss/keycloak
```

After the container is started, so we head to [http://localhost:9001](http://localhost:9001)
(or the respectful port, if one changed the mapping above), and log in using
`admin` as the username and the password.

### Create a new realm

To create a new realm, simply hover over the current realm in the top right
corner, and click on the blue "Add realm" button.

![New realm location]({{ site.baseurl }}public/images/new_realm_location.png)

Let's call it `spring-security-demo`, and create the realm.
![Create realm]({{ site.baseurl }}public/images/create_keycloak_realm.png)

### Create client with roles

First, we should verify that we are in the right realm (one should see
the new realm name `spring-security-demo` in the top right corner). 

### Create users

# The rest service
