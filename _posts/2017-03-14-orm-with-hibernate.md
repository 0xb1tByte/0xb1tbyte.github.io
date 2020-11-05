---
layout: post
title:  "ORM with Hibernate"
date:   2017-03-14 23:15:00 +0800
categories: java
tags: orm hibernate
---
## Goal

Using Hibernate as ORM implementation.

## Technique

1. [Hibernate][hibernate]
2. [c3p0][c3p0] - Substitute for Hibernate's build-in connection pools.
3. [Gradle][gradle]

[hibernate]: http://hibernate.org/orm/
[c3p0]: http://www.mchange.com/projects/c3p0/
[gradle]: https://gradle.org/

## IDE

- Eclipse with plugin - [JBoss Tools][jboss-tools]
  - [Hibernate tools][hibernate-tools] inside the Jboss Tools.

[jboss-tools]: http://tools.jboss.org/downloads/jbosstools/
[hibernate-tools]: https://tools.jboss.org/features/hibernate.html

## Step by step

### Configuration

1. `build.gradle`

    ```gradle
    dependencies {
        ...
        compile 'org.hibernate:hibernate-core:5.2.8.Final'
        compile 'org.hibernate:hibernate-c3p0:5.2.8.Final'
        ...
    }
    ```

2. ``hibernate.cfg.xml``

    ```xml
    <hibernate-configuration>
        <session-factory>
            ...
            <property name="hibernate.current_session_context_class">thread</property> //...(1)
            <property name="hibernate.connection.provider_class">org.hibernate.c3p0.internal.C3P0ConnectionProvider</property> //...(2)
            ...
        </session-factory>
    </hibernate-configuration>
    ```

    - (1): Set `thread` to enable `sessionFactory.getCurrentSession()`.
    - (2): Let **c3p0** to manage connection pools.

### Usage

- [Quick Start][hibernate-quick-start] - Using Native Hibernate APIs and Annotation Mappings.
- DAO Pattern
  - [Core J2EE Patterns - Data Access Object][dao-pattern]
  - [Generic Data Access Objects][generic-dao]

[hibernate-quick-start]: http://docs.jboss.org/hibernate/orm/current/quickstart/html_single/#tutorial_annotations
[dao-pattern]: http://www.oracle.com/technetwork/java/dataaccessobject-138824.html
[generic-dao]: https://developer.jboss.org/wiki/GenericDataAccessObjects

## Reference

- [Hibernate ORM User Guide][hibernate-user-guide]
- [Basic Types][basic-types]

[hibernate-user-guide]: https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html
[basic-types]: http://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#basic
