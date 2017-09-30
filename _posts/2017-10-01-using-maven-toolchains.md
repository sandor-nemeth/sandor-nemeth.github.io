---
published: false
layout: post
title: Using Maven toolchains
categories:
  - programming
---

This time I would like to show you a little but pretty neat functionality in Maven, which is called [toolchains]. The purpose of this feature is to provide configurable tooling for the builds. For example let's say that I have different projects requiring different major versions of the JDK. I could use the `maven-compiler-plugin` to set the source and target versions, but that will not really help me accidentally using some APIs which are not present in the actual JDK running the application later (like using the new Java 8 Date API for a project running on Java 7). 

For this problem toolchains are the solution. I can specify multiple JDKs, and tell Maven which one to use. In this short post I'll show how to do that. 

## Configuration of the environment

First a new file `$USER_HOME/.m2/toolchains.xml` needs to be created. This file describes where our tools are. For Java 8 and Java 9 this is how mine looks like:

```xml
<?xml version="1.0" encoding="UTF8" ?>
<toolchains>
    <toolchain>
        <type>jdk</type>
        <provides>
            <version>1.8</version>
            <vendor>oracle</vendor>
        </provides>
        <configuration>
            <jdkHome>/Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home</jdkHome>
        </configuration>
    </toolchain>
    <toolchain>
        <type>jdk</type>
        <provides>
            <version>1.9</version>
            <vendor>oracle</vendor>
        </provides>
        <configuration>
            <jdkHome>/Library/Java/JavaVirtualMachines/jdk-9.jdk/Contents/Home</jdkHome>
        </configuration>
    </toolchain>
</toolchains>
```

This essentially provides the 2 JDK versions we need.

And now let's set up maven to use it in our project: 




[toolchains]: https://maven.apache.org/guides/mini/guide-using-toolchains.html
