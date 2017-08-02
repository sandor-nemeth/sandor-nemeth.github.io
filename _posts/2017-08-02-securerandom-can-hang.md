---
layout: post
title: Java SecureRandom can hang on linux
date: 2017-08-02
categories: [java]
tags: [java]
---

Did you know that `java.security.SecureRandom` can hang indefinitely?
Me neither. But now I do! Yesterday I had a weird issue at work, where
one test kept hanging, but only when I ran it through Jenkins, and I
wasn't able to reproduce the problem locally. Oh yeah, you also love
these, or? Anyways, at first I had no idea what this could be, although
one of my collagues had a hunch that this is caused by the
random-generator used in the test.

Turns out that he was right. What happens is that Java initially relies
on the `/dev/random` as random number generator, which relies on the
usage of the system (e.g. moving the mouse) to generate random numbers.
If there is not enough noise, it will block until there is additional
noise generated.

> When the entropy pool is empty, reads from /dev/random will block until
  additional environmental noise is gathered.
  (Source: Linux Programmer's Manual, section 4)

Oh yeah. And the solution (from [here][1]): you have to use
`file:/dev/./urandom` as the source of `java.security.SecureRandom`.
This is achievable in 2 ways: either run the app with a system
parameter:

```java
-Djava.security.egd=file:/dev/./urandom
```

or set in the `securerandom.source` value in the `java.security` file
to the same value.

Bits of wisdom for today...

[1]: http://bugs.java.com/view_bug.do?bug_id=6521844
