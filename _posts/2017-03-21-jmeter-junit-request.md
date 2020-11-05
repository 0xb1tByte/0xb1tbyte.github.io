---
layout: post
title:  "JMeter with JUnit Request"
date:   2017-03-21 21:30:00 +0800
categories: java
tags: jemeter junit
---
## Goal

Do load test in custom request using JUnit Request provided by JMeter.

## Technique

1. [JMeter][jmeter]
2. [JUnit][junit] - with JUnit4 annotation-based development style.
3. [Gradle][gradle]

[jmeter]: http://jmeter.apache.org/
[junit]: http://junit.org/
[gradle]: https://gradle.org/

## About JMeter

A Test plan hierarchy is like below:

```text
JMeter
    |-Test Plan
    |-Threads Group
        |-Sampler      <-Use JUnit Request sampler.
        |-Listener
```

*Have a glimpse of offical JUnit Sampler turotial at [Reference](#reference)*.

## Step by step

Take previous post [gRPC in Java][grpc-in-java] as example,
if we want to test Server with multi-thread Clients?

### Installation

Download [JMeter][jmeter-download] and unzip it.

[jmeter-download]: http://jmeter.apache.org/download_jmeter

### Configuration

Add dependencies in `build.gradle` of Java project.

```gradle
dependencies {
    ...
    testCompile 'junit:junit:4.12'
    ...
}
```

### Development

1. Create a JUnit test case.

    ```java
    public class HelloWorldClientTest{

        private final String host = "localhost";
        private final int port = 50051;
        private HelloWorldClient client;

        public HelloWorldClientTest(){ //...(1)
        }

        public HelloWorldClientTest(String text){ //...(2)
        }

        @Before
        public void setUp() throws Exception {
            client = new HelloWorldClient(host, port);
        }

        @After
        public void tearDown() throws Exception {
            client = null;
        }

        @Test
        public void greet(){
            client.greet("Anonymous");  //...(3)
        }

    }
    ```

    - (1).(2): **Important!** Required constructors for JMeter's JUnit Request Sampler, or it will throw exception when instantiate test case.
    - (3): Send Request to Server.

2. Export Test case to a `JAR` file.

    *//e.g. helloworld_client_test.jar*

[grpc-in-java]: {{ site.baseurl }}{% post_url 2017-03-10-grpc-in-java %}

### Usage

1. Put `helloworld_client_test.jar` under JMeter directory:

    ```text
    .JMeter
    |-...
    |-bin
    |   |-jmeter.bat
    |   |-...
    |-...
    |-lib
    |   |-junit
    |         |-helloworld_client_test.jar  <- HERE!
    |-...
    ```

2. Start JMeter, execute `bin/jmeter.bat`.

3. Configure a Test Plan:
    1. Right Click on *Test Plan* -> [Add] -> [Threads] -> [Thread Group]
    2. Right Click on *Thread Group* -> [Add] -> [Sampler] -> [Junit request]
        - Check "Search for JUnit 4 annotations(instrad of JUnit 3)"
        - Select **Classname**: `com.....HelloWorldClientTest`
        - Select **Test Method**: `greet`
    3. Right Click on *Thread Group* -> [Add] -> [Listener] -> [View results tree]
    4. Select *Thread Group*: set **Thread Numbers**.

### Run

1. Run HelloWorldServer.
2. Press JMeter's Start button!

## Reference

- [JUnit Sampler][junit-sampler]: Offical JUnit Sampler Tutorial.

[junit-sampler]: http://jmeter.apache.org/usermanual/junitsampler_tutorial.pdf