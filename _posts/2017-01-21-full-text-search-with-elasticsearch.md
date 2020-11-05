---
layout: post
title:  "Traditional Chinese Full-text Search with Elasticsearch"
date:   2017-01-21 11:20:10 +0800
categories: java
tags: elasticsearch logstash
---
## Goal

Traditoinal Chinese Full-text search

## Technique

1. [Elasticsearch][elasticseach] + [Smart Chinese Analysis plugin][smartcn]
2. [Logstash][logstash] + [jdbc-plugin][jdbc-plugin]
3. Relational Database using [MariaDB][mariadb]

    [elasticseach]: https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html "Elasticsearch"
    [smartcn]: https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-smartcn.html "Smart Chinese Analysis plugin"
    [logstash]: https://www.elastic.co/guide/en/logstash/current/index.html "Logstash"
    [jdbc-plugin]: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html "jdbc-plugin"
    [mariadb]: https://mariadb.org/ "MariaDB"

## Flow

**Search** <-> Elasticsearch <- Logstash <- Relational Database

## Environment

- Windows 8
- Java8

## Step by Step

### Installation

1. Relational Database

    - Download and Install [MariaDB][mariadb-download]
    - Create db and table, example below:

        ```text
        [newspaper]
        news {
            id,
            title,
            content,
            create_date
        }
        ```

2. Elasticsearch

    - Download and extract [Elasticsearch][elasticseach-download]
    - Install Smart Chinese Analysis Plugin:

        ```shell
        $ bin/elasticsearch-plugin install analysis-smartcn
        ```

    [mariadb-download]: https://mariadb.org/download/ "MariaDB-Download"
    [elasticseach-download]: https://www.elastic.co/downloads/elasticsearch "Elasticsearch-Download"

3. Logstash

    - Download and extract [Logstash][logstash-download]
    - Jdbc-plugin may **integrated already**, check below:

        ```shell
        $ bin/logstash-plugin list

        ...
        logstash-input-imap
        logstash-input-irc
        logstash-input-jdbc <- HERE!
        logstash-input-kafka
        logstash-input-log4j
        ...
        ```

        (2017-01-21)
        if not, it will **fail** when installing jdbc-plugin manually in windows:

        ```shell
        $ bin/logstash-plugin install logstash-input-jdbc

        ...
        Encoding::UndefinedConversionError  <- ERROR!
        ...
        ```

        *My experience: Install failed -> feel depressed:( -> check plugin-list -> already exist -> ...

    [logstash-download]: https://www.elastic.co/downloads/logstash "Logstash-Download"

### Configuration

- Logstash
    1. Download Jdbc driver: [MariaDB-Connector][mariadb-connector]

        ```text
        #Dir hierarchy
        |-root
        |    |-bin
        |        |-logstash
        |        |-mariadb-java-client-x.x.x.jar
        ```

    2. Create configuration file: `logstash.conf`

        ```text
        #Dir hierarchy
        |-root
        |    |-bin
        |        |-logstash
        |        |-mariadb-java-client-x.x.x.jar
        |        |-logstash.conf
        ```

        ```config
        #logstash.conf
        input {
            jdbc {
                jdbc_driver_library => "mariadb-java-client-x.x.x.jar"
                jdbc_driver_class => "org.mariadb.jdbc.Driver"
                jdbc_connection_string => "jdbc:mariadb://localhost:3306/newspaper"
                jdbc_user => "username"
                jdbc_password => "password"
                jdbc_validate_connection => true
                schedule => "* * * * *"
                statement => "SELECT * from news"
            }
        }
        output {
            elasticsearch {
                index => "news"
                document_type => "new"
                document_id => "%{id}"
            }
        }
        ```

    [mariadb-connector]: https://downloads.mariadb.org/connector-java/ "MariaDB-Connector"

### Run

1. Start Elasticsearch

    ```shell
    $ bin/elasticsearch
    ```

2. Start Logstash

    ```shell
    $ bin/logstash -f logstash.conf
    ```

3. Check data with browser:

    <http://localhost:9200/news/>

4. Finally, **search** !

    <http://localhost:9200/news/_search?q=keyword&pretty>

    *`pretty` parameter, tells Elasticsearch to return pretty-printed JSON results.

## Reference

1. [Elastic full-text search][elastic-full-text-search]
2. [Logstash jdbc example][logstash-jdbc-example]

    [elastic-full-text-search]: https://www.elastic.co/guide/en/elasticsearch/guide/current/full-text-search.html "elastic-full-text-search"
    [logstash-jdbc-example]: https://www.elastic.co/blog/logstash-jdbc-input-plugin "Logstash-jdbc-example"
