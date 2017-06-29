---
layout: post
title: Secure a Spring Boot Rest app with Spring Security and Keycloak
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

After this, we just simply log in to the container and navigate to the `bin`
folder.

```bash
docker exec -it keycloak /bin/bash
cd keycloak/bin
```

First of all, we need to log in to the keycloak server from the CLI client,
and afterwards we will not need any more authentication:

```bash
$ ./kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user admin --password admin
```

### Configure the realm

First, we need to create a realm, so let's do just that:

```bash
$ ./kcadm.sh create realms -s realm=spring-security-example -s enabled=true
```

Afterwards, we need to create 2 clients, which will provide authentication for
our applications. First we create a cURL client, so we can log in via a
command line command:

```bash
$ ./kcadm.sh create clients -r spring-security-example -s clientId=curl -s enabled=true -s publicClient=true -s baseUrl=http://localhost:8080 -s adminUrl=http://localhost:8080 -s directAccessGrantsEnabled=true
```

It is important to notice 2 options here: `publicClient=true` and
`directAccessGrantsEnabled=true`. The first one makes this client public, which
means that our cURL client can initiate a login without providing any secret.
The second one enables us to log in directly using the username and password.

And secondly we create a client which is used by our REST service:

```bash
$ ./kcadm.sh create clients -r spring-security-example -s clientId=spring-security-demo-app -s enabled=true -s baseUrl=http://localhost:8080 -s bearerOnly=true
```

Here the important configuration is `bearerOnly=true`. This tells Keycloak that
the client never initiates a login process, but when it receives a `Bearer`
token, then it will check the validity of said token.

Both command will output something like:

```bash
Created new client with id '607cae5a-2962-4c0e-945e-915ceafc17f5'
```

We should note these IDs, because we will use them in the next steps.

So we have the two clients, and next up is to create roles for the
`spring-security-demo-app` client:

```bash
$ ./kcadm.sh create clients/607cae5a-2962-4c0e-945e-915ceafc17f5/roles -r spring-security-example -s name=admin -s 'description=Admin role'
$ ./kcadm.sh create clients/607cae5a-2962-4c0e-945e-915ceafc17f5/roles -r spring-security-example -s name=user -s 'description=User role'
```

Finally we should get the configuration for the client, to provide it later
to our application:

```bash
$ ./kcadm.sh  get clients/607cae5a-2962-4c0e-945e-915ceafc17f5/installation/providers/keycloak-oidc-keycloak-json -r spring-security-example
```

Which should return something similar to this:

```json
{
  "realm" : "spring-security-example",
  "bearer-only" : true,
  "auth-server-url" : "http://localhost:8080/auth",
  "ssl-required" : "external",
  "resource" : "spring-security-demo-app",
  "use-resource-role-mappings" : true
}
```

And with this we should be fully prepared to create our users.



### Configure the users

For the demo purposes, we should create 2 users with 2 different roles, so we
can verify that the authorization works.

First, let's create a user having the `admin` role:

```bash
$ ./kcadm.sh create users -r spring-security-example -s username=joe_admin -s enabled=true
Created new user with id '7a0b80c1-fd3a-48a3-b498-c6f78121d284'
$ ./kcadm.sh update users/7a0b80c1-fd3a-48a3-b498-c6f78121d284/reset-password -r spring-security-example -s type=password -s value=admin -s temporary=false -n
$ ./kcadm.sh add-roles -r spring-security-example --uusername=joe_admin --cclientid spring-security-demo-app --rolename admin
```

In the snippet above, first we created the user (with `create users`), then we
set a password (with `update`), and added the user to the `admin` role.

**Note: never use this method in production, it is only for demonstration purposes!**

Then we create another user, this time having the role `user`:

```bash
$ ./kcadm.sh create users -r spring-security-example -s username=jim_user -s enabled=true
Created new user with id '213f2d11-0bfb-4d76-a3e4-7daf02117692'
$ ./kcadm.sh update users/213f2d11-0bfb-4d76-a3e4-7daf02117692/reset-password -r spring-security-example -s type=password -s value=admin -s temporary=false -n
$ ./kcadm.sh add-roles -r spring-security-example --uusername=jim_user --cclientid spring-security-demo-app --rolename user
```

# The rest service

Now that we have Keycloak configured, and ready to use, we just need an app to
utilize it! So we create a simple Spring Boot application. I'll use
[gradle](https://gradle.org/) here:

```gradle
plugins {
    id 'java'
    id 'idea'
    id 'org.springframework.boot' version '1.5.4.RELEASE'
}

sourceCompatibility = '1.8'
targetCompatibility = '1.8'

repositories {
    jcenter()
}

dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web'
    compile 'org.springframework.boot:spring-boot-starter-security'

    compile 'org.keycloak:keycloak-spring-boot-starter:3.1.0.Final'
    compile 'org.keycloak:keycloak-spring-security-adapter:3.1.0.Final'

    testCompile 'org.springframework.boot:spring-boot-starter-test'
}
```

This adds all needed dependencies:

- `spring-security` for securing the application
- `keycloak-spring-boot-starter` for using Keycloak with Spring Boot
- `keycloak-spring-security-adapter` for integrating with Spring Security

We need a simple application class:

```java
package io.github.sandornemeth;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class KeycloakSecurityDemoApp {

    public static void main(final String... args) {
        SpringApplication.run(KeycloakSecurityDemoApp.class, args);
    }

}
```

And an endpoint where we say hello to our selected users:

```java
package io.github.sandornemeth;

import org.springframework.security.access.annotation.Secured;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloEndpoint {

    @GetMapping("/admin/hello")
    @Secured("ROLE_ADMIN")
    public String sayHelloToAdmin() {
        return "Hello Admin";
    }

    @GetMapping("/user/hello")
    @Secured("ROLE_USER")
    public String sayHelloToUser() {
        return "Hello User";
    }

}
```

And finally the keycloak configuration:

```java
package io.github.sandornemeth;

import org.keycloak.adapters.KeycloakConfigResolver;
import org.keycloak.adapters.springboot.KeycloakSpringBootConfigResolver;
import org.keycloak.adapters.springsecurity.authentication.KeycloakAuthenticationProvider;
import org.keycloak.adapters.springsecurity.config.KeycloakWebSecurityConfigurerAdapter;
import org.keycloak.adapters.springsecurity.filter.KeycloakAuthenticationProcessingFilter;
import org.keycloak.adapters.springsecurity.filter.KeycloakPreAuthActionsFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.authority.mapping.GrantedAuthoritiesMapper;
import org.springframework.security.core.authority.mapping.SimpleAuthorityMapper;
import org.springframework.security.web.authentication.session.NullAuthenticatedSessionStrategy;
import org.springframework.security.web.authentication.session.SessionAuthenticationStrategy;

@Configuration
@EnableWebSecurity
public class KeycloakSecurityConfigurer extends KeycloakWebSecurityConfigurerAdapter {

  @Bean
  public GrantedAuthoritiesMapper grantedAuthoritiesMapper() {
    SimpleAuthorityMapper mapper = new SimpleAuthorityMapper();
    mapper.setConvertToUpperCase(true);
    return mapper;
  }

  @Override
  protected KeycloakAuthenticationProvider keycloakAuthenticationProvider() {
    final KeycloakAuthenticationProvider provider = super.keycloakAuthenticationProvider();
    provider.setGrantedAuthoritiesMapper(grantedAuthoritiesMapper());
    return provider;
  }

  @Override
  protected void configure(final AuthenticationManagerBuilder auth) throws Exception {
    auth.authenticationProvider(keycloakAuthenticationProvider());
  }

  @Override
  protected SessionAuthenticationStrategy sessionAuthenticationStrategy() {
    return new NullAuthenticatedSessionStrategy();
  }

  @Override
  protected void configure(final HttpSecurity http) throws Exception {
    super.configure(http);
    http
        .authorizeRequests()
        .antMatchers("/admin/*").hasRole("ADMIN")
        .antMatchers("/user/*").hasRole("USER")
        .anyRequest().permitAll();
  }

  @Bean
  KeycloakConfigResolver keycloakConfigResolver() {
    return new KeycloakSpringBootConfigResolver();
  }

  @Bean
  public FilterRegistrationBean keycloakAuthenticationProcessingFilterRegistrationBean(
      final KeycloakAuthenticationProcessingFilter filter) {
    final FilterRegistrationBean registrationBean = new FilterRegistrationBean(filter);
    registrationBean.setEnabled(false);
    return registrationBean;
  }

  @Bean
  public FilterRegistrationBean keycloakPreAuthActionsFilterRegistrationBean(
      final KeycloakPreAuthActionsFilter filter) {
    final FilterRegistrationBean registrationBean = new FilterRegistrationBean(filter);
    registrationBean.setEnabled(false);
    return registrationBean;
  }
}
```

OK, so there is a lot of stuff in this configuration, so let's walk over it.

First of all, the `KeycloakSecurityConfigurer` class extends
`KeycloakWebSecurityConfigurerAdapter`, which is a class provided by Keycloak
that provides integration with Spring Security.

Then we configure the authentication manager with the addition of a
[`SimpleAuthorityMapper`](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/core/authority/mapping/SimpleAuthorityMapper.html),
which is responsible for converting the role name
coming from Keycloak to match the conventions of Spring Security. Basically
Spring Security expects rolenames to start with the `ROLE_` prefix, and
we have 2 choices: either we name our roles like `ROLE_ADMIN` in Keycloak,
or we can name them like `admin`, and then use this mapper to convert it
to uppercase and prepend the necessary `ROLE_` prefix:

```java
@Bean
public GrantedAuthoritiesMapper grantedAuthoritiesMapper() {
  SimpleAuthorityMapper mapper = new SimpleAuthorityMapper();
  mapper.setConvertToUpperCase(true);
  return mapper;
}

@Override
protected KeycloakAuthenticationProvider keycloakAuthenticationProvider() {
  final KeycloakAuthenticationProvider provider = super.keycloakAuthenticationProvider();
  provider.setGrantedAuthoritiesMapper(grantedAuthoritiesMapper());
  return provider;
}

@Override
protected void configure(final AuthenticationManagerBuilder auth) throws Exception {
  auth.authenticationProvider(keycloakAuthenticationProvider());
}
```

We also need to set a session strategy for Keycloak, but as we are creating a
stateless REST service we do not really want to have sessions, therefore we
utilize the `NullAuthenticatedSessionStrategy`:

```java
@Override
protected SessionAuthenticationStrategy sessionAuthenticationStrategy() {
  return new NullAuthenticatedSessionStrategy();
}
```

Normally, the Keycloak Spring Security integration resolves the keycloak
configuration from a `keycloak.json` file, however we would like to have
proper Spring Boot configuration, so we override the configuration resolver
with the one for Spring Boot:

```java
@Bean
KeycloakConfigResolver keycloakConfigResolver() {
  return new KeycloakSpringBootConfigResolver();
}
```

And then we configure Spring Security to authorize all requests:

```java
@Override
protected void configure(final HttpSecurity http) throws Exception {
  super.configure(http);
  http
      .authorizeRequests()
      .anyRequest().permitAll();
}
```

And finally, per documentation we prevent double-registering the filters
for Keycloak:

```java
@Bean
public FilterRegistrationBean keycloakAuthenticationProcessingFilterRegistrationBean(
    final KeycloakAuthenticationProcessingFilter filter) {
  final FilterRegistrationBean registrationBean = new FilterRegistrationBean(filter);
  registrationBean.setEnabled(false);
  return registrationBean;
}

@Bean
public FilterRegistrationBean keycloakPreAuthActionsFilterRegistrationBean(
    final KeycloakPreAuthActionsFilter filter) {
  final FilterRegistrationBean registrationBean = new FilterRegistrationBean(filter);
  registrationBean.setEnabled(false);
  return registrationBean;
}
```

And lastly we need to configure our application in the
`application.properties` with the values we downloaded earlier:

```
server.port=9002

keycloak.realm = spring-security-example
keycloak.bearer-only = true
keycloak.auth-server-url = http://localhost:9001/auth
keycloak.ssl-required = external
keycloak.resource = spring-security-demo-app
keycloak.use-resource-role-mappings = true
keycloak.principal-attribute = preferred_username
```

# Using the application

So now we are fully configured, let's see if everything works as we expected.

First run the application:

```bash
$ gradle bootRun
```

Then authenticate with the `curl` client we created, to get the access token:

```bash
$ export TOKEN=`curl -ss --data "grant_type=password&client_id=curl&username=joe_admin&password=admin" http://localhost:9001/auth/realms/spring-security-example/protocol/openid-connect/token | jq -r .access_token`
```

This will store the access token we receive in the `TOKEN` variable.

And now we can check that our admin can access his own `/admin/hello` endpoint:

```bash
$ curl -H "Authorization: bearer $TOKEN" http://localhost:9002/admin/hello
Hello Admin
```

but it cannot access the endpoint `/user/hello`:

```bash
$ curl -H "Authorization: bearer $TOKEN" http://localhost:9002/user/hello
{"timestamp":1498728302626,"status":403,"error":"Forbidden","message":"Access is denied","path":"/user/hello"}
```

And the same is true for the other user.

## References

Here are the links for all the material I was using when writing this article:

- [Keycloak admin CLI documentation](https://keycloak.gitbooks.io/documentation/server_admin/topics/admin-cli.html)
- [Keycloak Spring Boot adapter](https://keycloak.gitbooks.io/documentation/securing_apps/topics/oidc/java/spring-boot-adapter.html)
- [Keycloak Spring Security adapter](https://keycloak.gitbooks.io/documentation/securing_apps/topics/oidc/java/spring-security-adapter.html)

Source code is on [Github](https://github.com/sandor-nemeth/spring-security-rest-keycloak)
