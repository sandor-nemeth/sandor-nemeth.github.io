---
published: false
layout: default
categories: java
---
# Diving into JUnit5

[JUnit 5][1] got the first GA release a few days back, and I wanted to take a look on it, to see what are the new things, and how can I integrate it into my everyday work life.

Warning: this is a long post!

## Dependencies - Let's use JUnit 5!

Setting up the configuration requires a bit more work than before, because as of now there is no automatic support in the Maven Surefire plugin for picking up these tests, which means that one has to configure this support on its own. There is [an issue][2] for this in the Surefire JIRA though, so hopefully at some point this inconvenience will be resolved. Also, JUnit 5 requires Java 8 to run, which is also something we'll configure. 

```xml
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>


    <junit.jupiter.version>5.0.0</junit.jupiter.version>
    <junit.platform.version>1.0.0</junit.platform.version>

    <logback.version>1.2.3</logback.version>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.6.1</version>
            <configuration>
                <source>${maven.compiler.source}</source>
                <target>${maven.compiler.target}</target>
                <encoding>${project.build.sourceEncoding}</encoding>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.20.1</version>
            <configuration>
                <properties>
                    <excludeTags>slow</excludeTags>
                </properties>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>${junit.platform.version}</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

## Writing tests



### General changes and t

### Tagging and filtering

### Nested tests

### Parameterized tests

### Generating tests

### Extensions

## Running tests


[1]: http://junit.org/junit5/
[2]: https://issues.apache.org/jira/browse/SUREFIRE-1206
