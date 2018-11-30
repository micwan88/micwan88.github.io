---
title: Load class or resource file from bytes in Java
categories: java classloader
---

## Load class or resource file from bytes array in Java

We can extend `ClassLoader` class to implement a custom class loader so that we can load class / resource file from bytes array. Before that, we need to know the hierarchy architecture and its loading sequence of classloader. Otherwise, we do not know which method in `ClassLoader` should be overrided.

### ClassLoader hierarchy and loading sequence

- For Java Class

Below is the sequence of how Java look up a class:

``` java
CustomClassLoader.loadClass() -> Parent.loadClass() -> Parent.Parent.loadClass() -> ... -> 
No more parent then BootstrapClassLoader load class -> Parent....Parent.findClass() -> ... ->
Parent.Parent.findClass() -> Parent.findClass() -> CustomClassLoader.findClass() -> ClassNotFoundException
```


According to the above sequence, you can see all classloader will try look up the class from parent first. If the class cannot be found by its parent, it will try load the class by `findClass()` itself. If it is failed again, throw a `ClassNotFoundException` directly.

So if we want to implement a classloader to use bytes array, we should override the method of `findClass()` and it is enough.

- For Resource File

While we are calling `getResourceAsStream()` from any classloader, actually it will try look up the resources from its parent first and it just same as loading Java class. Below is the sequence of how Java look up a resource name.

``` java
//Classloader retrieve the resource file as URL first, then calling URL.openStream()
CustomClassLoader.getResourceAsStream() -> ((URL)CustomClassLoader.getResource()).openStream()

//Here is the resource file (URL) loop up sequence
CustomClassLoader.getResource() -> Parent.getResource() -> Parent.Parent.getResource() -> ... -> 
No more parent then BootstrapClassLoader get resource -> Parent....Parent.findResource() -> ... ->
Parent.Parent.findResource() -> Parent.findResource() -> CustomClassLoader.findResource() -> null
```


If the resource file cannot be found by its parent, it will try load the resource file by `findResource()` itself. If it is failed again, return null directly.

So if we want to implement a classloader to use bytes array for resource file, we should override the method of `findResource()` and it is enough.

References:
- [Class Loaders in Java](https://www.baeldung.com/java-classloaders)
- [Any way to create a URL from a byte array?](https://stackoverflow.com/questions/17776884/any-way-to-create-a-url-from-a-byte-array)
- [Writing a Protocol Handler](https://www.oreilly.com/library/view/learning-java/1565927184/apas02.html)
