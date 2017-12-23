---
title: Logging configuration from env variables in Spring Boot
---
Today's big learning experience was with Spring Boot logging configuration. We wanted to adjust the configured logging level via an environment variable. As it is normally configured via a property in the syntax of `logging.level.com.acme=DEBUG`, we tried to use the normal environmental conversion of this name. Namingly: 

```bash
LOGGING_LEVEL_COM_ACME=DEBUG
```

And it got ignored. Which was weird, because normally all properties can be overriden from environment variables. And it is not possible. Like ever.

So the only option we have for now is to move back to Java arguments:

```bash
-Dlogging.level.com.acme=DEBUG
```

Which is not so funny, and not something that I was expecting from Spring Boot. 
