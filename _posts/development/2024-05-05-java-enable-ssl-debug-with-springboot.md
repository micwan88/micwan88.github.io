---
title: Java enable SSL debug (include spring-boot:run)
categories: java springboot
---

- JVM Argument (non-springboot:run)

For Java program which is not calling via `spring-boot:run`, you can add JVM argument `-Djavax.net.debug=ssl:handshake:keymanager:trustmanager` to enable the SSL debug.

- For spring-boot:run program

For spring-boot:run program, you can add argument `-Dspring-boot.run.jvmArguments="-Djavax.net.debug=ssl:handshake:keymanager:trustmanager"` to enable the SSL debug or you can declare it inside code as below.

``` java

System.setProperty("javax.net.debug", "ssl:handshake:keymanager:trustmanager")

```

References:
- [Debugging SSL/TLS Connections](https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#Debug)
- [Enabling SSL debugging in a standalone Java program](https://access.redhat.com/solutions/973783)
- [Maven spring boot run debug with arguments](https://stackoverflow.com/questions/36217949/maven-spring-boot-run-debug-with-arguments)
