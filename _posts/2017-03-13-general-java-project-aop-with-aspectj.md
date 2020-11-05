---
layout: post
title:  "General Java project AOP with AspectJ"
date:   2017-03-13 17:53:00 +0800
categories: java
tags: aop aspectj
---
## Goal

Implement AOP in general Java project with AspectJ.

## Technique

1. [AspectJ][aspectj] - with annotation-based development style.

[aspectj]: https://eclipse.org/aspectj/

## IDE & Tool

1. [Eclipse][eclipse]
2. [AJDT][ajdt]

[eclipse]: https://eclipse.org/
[ajdt]: http://www.eclipse.org/ajdt/

## AspectJ Basic Concept

> "Pointcut" pick out "JoinPoint" then execute "Advice".

1. [JoinPoint][joinpoint]
    - Many kinds of [JoinPoints][kinds-of-joinpoint]
2. [Pointcut][pointcut]
    - [Pointcuts][pointcut-define] provided by AspectJ.
3. [Advice][advice]
    - [Advices][advices] to use.

[joinpoint]: https://eclipse.org/aspectj/doc/released/progguide/starting-aspectj.html#the-dynamic-join-point-model
[kinds-of-joinpoint]: https://eclipse.org/aspectj/doc/released/progguide/semantics-joinPoints.html
[pointcut]: https://eclipse.org/aspectj/doc/released/progguide/starting-aspectj.html#pointcuts
[pointcut-define]: https://eclipse.org/aspectj/doc/released/progguide/semantics-pointcuts.html
[advice]: https://eclipse.org/aspectj/doc/released/progguide/starting-aspectj.html#advice
[advices]: https://eclipse.org/aspectj/doc/released/progguide/starting-aspectj.html#advices

## Step by step

### Installation

1. Pick a **Update Site URL** of AJDT from download page: [AJDT-Download][ajdt-download]

    ```text
    e.g.
    [Development builds for Eclipse 4.6]
    http://download.eclipse.org/tools/ajdt/46/dev/update
    ```

2. In Eclipse, [Help] -> [Install New software] -> [Add]:

    ```text
    Name: ____
    Location: http://download.eclipse.org/tools/ajdt/46/dev/update
    ```

    -> Check **AspectJ Development Tool(Required)** -> [Finish]

    [ajdt-download]: http://www.eclipse.org/ajdt/downloads/

### Configuration

- If ( Java project **Exist** ):

    Right Click on Project -> [Configuration] -> [Convert to AspectJ Project].

- Else ( Create **New** AspectJ ):

    [File] -> [New] -> [Other] -> [AspectJ Project].

### Development

- Simple example below:

    ```java
    @Aspect //...(1)
    public class ServiceAspect {

        @Pointcut("execution(* com.example.service.Service.doSomething())") //...(2)
        private final void doSomething(){}; //...(3)

        @Before("doSomething()") //...(4)
        public final void beforeDo(JoinPoint joinpoint){ //...(5)
            System.out.println("@Before doSomething(): " + joinpoint.getSourceLocation());
        }

        @After("doSomething()") //...(6)
        public final void afterDo(JoinPoint joinpoint){
            System.out.println("@After doSomething(): " + joinpoint.getSourceLocation());
        }

    }
    ```

  - (1): Annotate class as a **Aspect**.
  - (2): Define **Joinpoint** for **Pointcut**.
  - (3): Define **Pointcut** name.
  - (4): Execute **Advice**-"*Before*" JoinPoint named `doSomething()`.
  - (5): Inject JoinPoint through parameter so that we can call `joinpoint.getSourceLocation()`.
  - (6): Execute **Advice**-"*After*" JoinPoint named `doSomething()`.

### Run

- Run as -> [AspectJ/Java Application]

## Reference

- [The AspectJ Programming Guide][aspectj-guide]
- [The AspectJ 5 Development Kit Developer's Notebook][aspectj5] - annotation-based development style
- [Some Example Pointcuts][pointcut-example]

[aspectj-guide]: https://eclipse.org/aspectj/doc/released/progguide/
[aspectj5]: https://eclipse.org/aspectj/doc/released/adk15notebook/
[pointcut-example]: https://eclipse.org/aspectj/doc/released/progguide/language-joinPoints.html#some-example-pointcuts