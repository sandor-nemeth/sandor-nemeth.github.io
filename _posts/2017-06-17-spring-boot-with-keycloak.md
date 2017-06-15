---
layout: post
title: Secure a Spring Boot Rest app with Spring Security and Keycloak
categories: java keycloak authentication spring
---

So today I wanted to explore [Keycloak](http://www.keycloak.org/), and decided
to set up a very simple Spring Boot microservice which handles authentication
and authorization with Spring Security, using Keycloak as my authentication
source.

# Step 1: Configure Keycloak

```bash
docker run -d --name keycloak -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin -p 9001:8080 jboss/keycloak
```

Configure steps:

1. Create realm `spring-sample`
2. Create groups: `users`, `admins`
3. Create users: `user`, `admin`
4. Create client: `spring-sample-app`
  - Access type: Bearer-Only
