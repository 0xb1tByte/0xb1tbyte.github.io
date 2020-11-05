---
layout: post
title:  "gRPC in Java"
date:   2017-03-10 11:35:10 +0800
categories: java
tags: grpc
---
## Goal

Build a Client/Server service in Java using Protocol Buffers.

## Technique

1. [gRPC][grpc]
2. [Protocol Buffers][protobuf]
3. [Gradle][gradle]

    - [grpc/grpc-java][grpc-java]
    - [google/protobuf-gradle-plugin][protobuf-gradle-plugin]

    [grpc]: http://www.grpc.io/docs/guides/ "gRPC Guides"
    [protobuf]: https://developers.google.com/protocol-buffers/docs/overview "Protocol Buffers Developer Guide"
    [gradle]: https://gradle.org/ "Gradle"
    [grpc-java]: https://github.com/grpc/grpc-java "grpc-java"
    [protobuf-gradle-plugin]: https://github.com/google/protobuf-gradle-plugin "protobuf-gradle-plugin"

## What is RPC?

> Client execute a procedure as local, but call Server to do the procedure behind.

RPC - [Remote Procedure Call][wikipedia-rpc]

> Client-> Client.procedure <-> Server <-> Server.procedure

[wikipedia-rpc]: https://en.wikipedia.org/wiki/Remote_procedure_call "Wikipedia - RPC"

## Step by step

Using gRPC basic **hello world example** as tutorial.

### Configuration

1. Define gRPC services and Protocol Buffers
    - [helloworld.proto][hello-world-proto]

        - (Optional) Edit `option java_package = "...";` inside the proto.
        - (Optional) Edit `option java_multiple_files = false;` inside the proto.
        - Place `helloworld.proto` at anywhere.

            ```text
            e.g.
            .JavaProject
            |-proto
            |     |-helloworld.proto
            |     |-*.proto
            ```

    [hello-world-proto]: https://github.com/grpc/grpc-java/blob/master/examples/src/main/proto/helloworld.proto "helloworld.proto"

2. Configure `build.gradle` to develop gRPC and compile `*.proto`

    See: [grpc/grpc-java][grpc-java]

    ```gradle
    project.ext {
        protoSrcDir = './proto' //...(1)
        protoGeneratedBaseDir = "${projectDir}/src/" //...(2)
    }

    apply plugin: 'java'
    apply plugin: 'com.google.protobuf'

    buildscript { //...(3)  
        repositories {
            mavenCentral()
            maven {
                url "https://maven.eveoh.nl/content/repositories/releases"
            }
        }

        dependencies {
            classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.1'
        }
    }

    repositories {  
        mavenCentral()
        jcenter()
    }

    dependencies {  //...(4)
        compile 'io.grpc:grpc-netty:1.1.2'
        compile 'io.grpc:grpc-protobuf:1.1.2'
        compile 'io.grpc:grpc-stub:1.1.2'
    }

    sourceSets {
        main {
            proto {
                srcDir protoSrcDir  //...(5)
            }
        }

        test {
            proto {
                srcDir protoSrcDir  //...(6)
            }
        }
    }

    protobuf {  
        protoc {
            artifact = "com.google.protobuf:protoc:3.2.0"
        }

        plugins {
            grpc {  //...(7)
            artifact = 'io.grpc:protoc-gen-grpc-java:1.1.2'
            }
        }

        generateProtoTasks {
            all().each { task ->
                task.plugins {
                    grpc {
                        outputSubDir = 'java' //...(8)
                    }
                }
            }
        }

        generatedFilesBaseDir = protoGeneratedBaseDir //...(9)
    }
    ```

    - (1): Define `*.proto` source directoty.
    - (2): Define `*.proto` compiled files (*.java) directoty.
    - (3): Use external libraries for Gradle, see: [External dependencies for the build script][buildscript]
    - (4): Add [grpc-java][grpc-java] dependencies.
    - (5): Set `*.proto` source directoty. (Default: "src/main/proto")
    - (6): Set `*.proto` source directoty for test. (Default: "src/test/proto")
    - (7): Add grpc-plugin to protobuf compiler.
    - (8): Set gRPC output Directory.
        - output path:

            ```text
            $protoGeneratedBaseDir/main/$outputSubDir/$java_package/
            ```

            *//"main" path is added by default.*

        - **Hint:** grpc-plugin will compile **`service {}`** in `*.proto`

    - (9): Set `*.proto` output Directory.
        - output path:  

            ```text
            $protoGeneratedBaseDir/main/$java_package/
            ```

            *//"main" path is added by default.*

        - **Hint:** protobuf compiler will compile **`message {}`** in `*.proto`

    [buildscript]: https://docs.gradle.org/current/userguide/organizing_build_logic.html#sec:build_script_external_dependencies "build_script_external_dependencies"

### Take a break!

- Now, files will like below:

    ```text
    //if (helloworld.proto: option java_multiple_files = true);
    .
    |-src
    |   |-main
    |        |-java
    |             |-io
    |                |-grpc
    |                     |-examples
    |                             |-helloworld
    |                                 |-GreeterGrpc.java
    |                                 |-HelloReply.java
    |                                 |-HelloReplyOrBuilder.java
    |                                 |-HelloRequest.java
    |                                 |-HelloRequestOrBuilder.java
    |                                 |-HelloWorldProto.java
    |
    |-proto
    |   |-helloworld.proto
    ```

### Development

1. Creating Server

    [HelloWorldServer.java][helloworld_server.java]

    [helloworld_server.java]: https://github.com/grpc/grpc-java/blob/master/examples/src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java "HelloWorldServer.java"

2. Creating Client

    [HelloWorldClient.java][helloworld_client.java]

    [helloworld_client.java]: https://github.com/grpc/grpc-java/blob/master/examples/src/main/java/io/grpc/examples/helloworld/HelloWorldClient.java "HelloWorldClient.java"

### Run

- Run it!