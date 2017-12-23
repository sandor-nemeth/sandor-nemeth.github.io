---
published: true
title: Integration tests with spring boot - ClassNotFoundException
---
Seems like today is that day of the week. There was another pretty weird issue (before the [logging configuration], namingly I had some problems with running integration tests with the [maven-failsafe-plugin] for Spring Boot 1.5.7. I have the 2.20 failsafe plugin configured, and I got this error: 

```bash
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.acme.UserManagementServiceIT
[ERROR] Tests run: 1, Failures: 0, Errors: 1, Skipped: 0, Time elapsed: 0.376 s <<< FAILURE! - in com.acme.UserManagementServiceIT
[ERROR] initializationError(com.acme.UserManagementServiceIT)  Time elapsed: 0.008 s  <<< ERROR!
java.lang.NoClassDefFoundError: com/acme/persistence/entity/UserEntity
Caused by: java.lang.ClassNotFoundException: com.acme.persistence.entity.UserEntity
```

And I knew that the class was there, so I - and also my boss - just looked like dumb, because we couldn't comprehend initially what could go wrong there.

Turns out, that the guys at Surefire made a change from 2.18.1 to 2.19, namingly that they load the JAR file itself, instead of using the folders in `target/` especially the `target/classes` folder. In the same time, around 1.4 Spring also changed how they are packaging the applications, and now when you run the `spring-boot-maven-plugin`'s `repackage` goal, you'll end up having a JAR structure like this: 

```bash
- BOOT-INF
  |- classes
     |- com/acme/YourClass.class  
  |- lib
     |- a.jar
     |- b.jar
- META-INF
  |- maven/{groupId}/{artifactId}
  	 |- pom.xml
     |- pom.properties
  |- MANIFEST.MF
- org/springframework/boot/loader
```

But from this structure, the failsafe plugin has no chance to figure out what to load. Oh yeah, goody. 

So, what's the solution: Add the `target/classes` folder to the test classpath: 

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <configuration>
        <additionalClasspathElements>
            <additionalClasspathElement>${basedir}/target/classes</additionalClasspathElement>
        </additionalClasspathElements>
        <includes>
            <include>**/*IT.java</include>
        </includes>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

And everything works like a charm!

Happy coding!

[maven-failsafe-plugin]: http://maven.apache.org/surefire/maven-failsafe-plugin/
[logging configuration]: {{ site.baseurl }}{% post_url 2017-10-16-logging-config-in-spring-boot %}
