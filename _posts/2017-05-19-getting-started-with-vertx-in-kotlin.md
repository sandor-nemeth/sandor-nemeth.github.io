---
layout: post
title: Getting started with Vertx in Kotlin
---

This time I decided to take a good look at [Vertx](https://vertx.io), and as
I had a plan to learn [Kotlin](https://kotlinlang.org) anyways, I decided
to give them a spin together.

## Project setup

First let's just create a new Gradle project:

```bash
gradle init --type java-library
```

Then edit the `build.gradle` file to have kotlin and java support, along with
slf4j and logback support, targeting Java 8. At the end it should look similar
to this:

```gradle
plugins {
    id 'java'
    id 'idea'
    id 'application'
    id 'org.jetbrains.kotlin.jvm' version '1.1.2-2'
    id 'com.github.johnrengelman.shadow' version '2.0.0'

}

sourceCompatibility = '1.8'
targetCompatibility = '1.8'

mainClassName = "io.github.sandornemeth.abeona.AbeonaAppKt"

def vertx_version = '3.4.1'
def jackson_version = '2.8.7'
def logback_version = '1.2.2'

compileKotlin {
    kotlinOptions.jvmTarget = '1.8'
}

repositories {
    jcenter()
}

dependencies {
    // kotlin dependencies
    compile 'org.jetbrains.kotlin:kotlin-stdlib-jre8'
    compile 'org.jetbrains.kotlin:kotlin-reflect'

    // vertx
    compile "io.vertx:vertx-core:$vertx_version"
    compile "io.vertx:vertx-web:$vertx_version"

    // Jackson
    compile "com.fasterxml.jackson.core:jackson-databind:$jackson_version"
    compile "com.fasterxml.jackson.module:jackson-module-kotlin:$jackson_version"

    // Logging
    compile "ch.qos.logback:logback-classic:$logback_version"

    // Utilities
    compile 'org.javassist:javassist:3.21.0-GA'

    // Testing
    testCompile 'org.jetbrains.kotlin:kotlin-test'
    testCompile 'org.jetbrains.kotlin:kotlin-test-junit'
    testCompile 'junit:junit:4.12'
    testCompile "io.vertx:vertx-unit:$vertx_version"
    testCompile 'org.assertj:assertj-core:3.7.0'
}
```

I also use the `shadow` plugin from [johnrengelman/shadow](https://github.com/johnrengelman/shadow)
to package the complete application into a single fat-jar.

Now that we are all set up, let's get started with the application. First
create an entry point for the application:

```kotlin
fun main(args: Array<String>) {
    System.setProperty("vertx.logger-delegate-factory-class-name", SLF4JLogDelegateFactory::class.java.name)

    val vertx = Vertx.vertx()
    vertx.deployVerticle(ApiVerticle::class.java.name)
}
```

Here 2 things are happening:

- Setting the system property `vertx.logger-delegate-factory-class-name`
  enables us to use slf4j, and behind that, logback
- We create a new Vertx instance, and deploy our first verticle, which
  is responsible for handling our API requests.

Then create the verticle itself:

```kotlin
class ApiVerticle : AbstractVerticle() {

    companion object {
        val log = loggerFor(javaClass)
    }

    override fun start(startFuture: Future<Void>?) {
        val router = createRouter()
        val port = config().getInteger("http.port", 8080)

        vertx.createHttpServer()
                .requestHandler { router.accept(it) }
                .listen(port, { result ->
                    if (result.succeeded()) {
                        log.info("Listening on port $port")
                        startFuture?.complete()
                    } else {
                        startFuture?.fail(result.cause())
                    }
                })
    }

    private fun createRouter() = Router.router(vertx).apply {
        get("/").handler(handlerRoot)
    }

    val handlerRoot = Handler<RoutingContext> { req ->
        req.response().end("Hello world!")
    }
}
```

There are a couple of things happening here:

- first we create a _companion object_, which practically provides us static
  fields in Java
- Then we start the HTTP server, reading port from the configuration, or
  defaulting to `8080`, if it isn't configured
- lastly we create a `Router` instance, and handle a `GET` call to `/`
  where we just return `Hello world!`

And finally let't test the application. Run the build with `gradle build`,
and then run it: `java -jar build/libs/abeona-backend-all.jar`.

We can now send our first request:

```bash
# curl -XGET http://localhost:8080
Hello world!
```
