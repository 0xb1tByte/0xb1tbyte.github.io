---
layout: post
title:  "Logging framework :  SLF4J + Logback"
date:   2017-03-17 22:16:00 +0800
categories: java
tags: log
---
## Goal

Logging.

## Technique

1. [SLF4J][slf4j] - Logging Facade.
2. [Logback][logback] - Native implementations.
3. [Gradle][gradle]

[slf4j]: https://www.slf4j.org/
[logback]: https://logback.qos.ch/
[gradle]: https://gradle.org/

## Step by step

### Configuration

1. Add dependencies in `build.gradle`.

    ```gradle
    dependencies {
        ...
        compile 'org.slf4j:slf4j-api:1.7.21'
        compile 'ch.qos.logback:logback-classic:1.2.1'
        compile 'ch.qos.logback:logback-core:1.2.1'
        compile 'org.codehaus.groovy:groovy:2.4.9' //Enable configuration using groovy.
        ...
    }
    ```

2. Create ``logback.groovy`` file under java project to configure log pattern.

    ```groovy
    appender("STDOUT", ConsoleAppender) {
        encoder(PatternLayoutEncoder) {
            pattern = "%d{YYYY-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{15} - %msg %n"
        }
    }

    root(INFO, ["STDOUT"])
    ```

    See: [Groovy Configuration][groovy-configuration]

[groovy-configuration]: https://logback.qos.ch/manual/groovy.html

### Usage

- [Logback Introduction][logback-intro]

    Tips:

    ```java
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    ```

[logback-intro]: https://logback.qos.ch/manual/introduction.html