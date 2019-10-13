---
title: Load resource file by getResourceAsStream in static method of Java
categories: java
---

There are three methods to load resource file in classpath and they have some differences between them.

Below two methods will treat resources name as absolute path and without leading "/"


``` java
//1
SomeClass.class.getClassLoader().getResourceAsStream(resourcesName);

//2
Thread.currentThread().getContextClassLoader().getResourceAsStream(resourcesName);
```

Below method will treat resources name as absolute path if it have leading "/" or as relative path if no leading "/"

``` java
//3
SomeClass.class.getClass().getResourceAsStream(resourcesName);
```

## Load resource file by getResourceAsStream for webapp (from Application Server)

Sometimes static calling `SomeClass.class.getClassLoader().getResourceAsStream()` and `SomeClass.class.getClass().getResourceAsStream()` to load a resource file for webapp may not work properly as Application Server may use complex hierarchy ClassLoader for webapp. So the solution is as below:

``` java
//If call from static method
Thread.currentThread().getContextClassLoader().getResourceAsStream(resourcesName);

//If it is not from static method
this.getClass().getResourceAsStream(resourcesName);
```

References:
- [Classpath resources](https://www.javaworld.com/article/2077352/java-se/smartly-load-your-properties.html)
- [What is the difference between Class.getResource() and ClassLoader.getResource()](https://www.javaworld.com/article/2077404/core-java/got-resources-.html)
- [When should I use Thread.getContextClassLoader()](https://www.javaworld.com/article/2077344/core-java/find-a-way-out-of-the-classloader-maze.html)
- [Different ways of loading a file as an InputStream](https://stackoverflow.com/questions/676250/different-ways-of-loading-a-file-as-an-inputstream)