---
published: true
title: Dockerizing a Spring Boot app
layout: post
---

Today I wanted to summarize how to containerize Spring Boot applications. The
purpose is to build a production ready container which can be used in an app
deployed into a distributed / containerzed environment

- a reasonably small container
- make use of the container layer caching mechanism to reduce distribution size
  for different versions

## Spring Boot 2.3

When using Spring Boot 2.3, the simplest way to build a good container image is
to leverage the knowledge of the Spring core community, and use the support they
provide in the form of the [Cloud Native Buildpack] provided. Utilizing this
support is the most simple thing, because it is built into the Spring Boot
plugin both for Maven and Gradle, so one can just utilize the built in commands:

```bash
./gradlew bootBuildImage
```

or 

```bash
./mvnw spring-boot:build-image
```

If ther eis still a need for configuring your own build image, you can still
fall back to the methods available on the older versions.

## Spring Boot 2.2 and below

If the upgrade to Spring Boot 2.3 is not possible - or you want to just have
full control over your build process without relying on too many external
tooling, a Spring Boot application can be blown up manually to provide a
properly layered structure: 

Using Gradle:

```bash
mkdir -p build/dep && (cd $_ && jar -xf ../libs/*.jar)
```

Using Maven: 

```bash
mkdir -p target/dep && (cd target/dep && jar -xf ../*.jar)
```

And then the following Dockerfile would prepare a layered, reusable image:

```docker
FROM openjdk:14-alpine

ARG DEPENDENCY=build/dep

COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app

RUN addgroup -S app && adduser -S app -G app
USER app:app

ENTRYPOINT ["java","-cp","app:app/lib/*","io.github.sandornemeth.docker.DockerApplication"]
```

The only thing that is not included here is the
[cloudfoundry/java-buildpack-memory-calculator], which is used in the other
image above to calculate the memory configuration of the application. This
currently requires users of this image to manually configure the memory settings
and GC for containerized use.

[Cloud Native Buildpack]: https://spring.io/blog/2020/08/14/creating-efficient-docker-images-with-spring-boot-2-3
[cloudfoundry/java-buildpack-memory-calculator]: https://github.com/cloudfoundry/java-buildpack-memory-calculator