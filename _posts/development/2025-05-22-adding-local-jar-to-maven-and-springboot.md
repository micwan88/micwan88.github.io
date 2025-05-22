---
title: Adding local jar into maven and SpringBoot jar
categories: java maven spring-boot
---

To add a local jar file into maven project as the dependency, it almost same as other dependency in the `pom` file. The difference is just you have to declare the scope as `system` and provide the path of the jar file as below.


``` xml
<dependency>
    <groupId>micwan88.github.com</groupId>
    <artifactId>local-lib</artifactId>
    <version>1.0.0</version>
    <scope>system</scope>
    <systemPath>${basedir}/somewhere/local-lib-1.0.0.jar</systemPath>
</dependency>
```


Please noted that by default SpringBoot won't include this local jar when you build the SpringBoot executable jar.

To solve this problem, what you have to do is just adding one more setting for SpringBoot as below.


``` xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <includeSystemScope>true</includeSystemScope>
            </configuration>
        </plugin>
    </plugins>
</build>
```

References:
- [Adding local jar files to a Maven project](https://dev.to/wakeupmh/adding-local-jar-files-to-a-maven-project-1h9n)
