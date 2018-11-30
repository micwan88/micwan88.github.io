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


### For Java Class - override findClass() to use bytes array

In `ClassLoader`, there is a method can load Java class from bytes array and we can use it under `findClass()`.
Here is the example:


``` java
//Define Custom ClassLoader
public class ByteClassLoader extends ClassLoader {
	private HashMap<String, byte[]> byteDataMap = new HashMap<>();

	public ByteClassLoader(ClassLoader parent) {
		super(parent);
	}

	public void loadDataInBytes(byte[] byteData, String resourcesName) {
		byteDataMap.put(resourcesName, byteData);
	}

	@Override
	protected Class<?> findClass(String className) throws ClassNotFoundException {
		if (byteDataMap.isEmpty())
			throw new ClassNotFoundException("byte data is empty");
		
		String filePath = className.replaceAll("\\.", "/").concat(".class");
		byte[] extractedBytes = byteDataMap.get(filePath);
		if (extractedBytes == null)
			throw new ClassNotFoundException("Cannot find " + filePath + " in bytes");
		
		return defineClass(className, extractedBytes, 0, extractedBytes.length);
	}
}

//Example Usage
public static void main(String[] args) throws IOException {
	//prepare the bytes array
	byte[] byteData = .....;

	ByteClassLoader byteClassLoader = new ByteClassLoader(this.getClass().getClassLoader());
	//Load bytes into hashmap
	byteClassLoader.loadDataInBytes(byteData, "class.name.in.full.package");

	Class<?> helloWorldClass = byteClassLoader.loadClass("class.name.in.full.package");
}
```


By implementing the custom classloader as the above, you can load normal / other class from parent classloader as well (i.e. without lose its hierarchy loading feature).


### For Resource File - override findResource() to use bytes array




References:
- [Class Loaders in Java](https://www.baeldung.com/java-classloaders)
- [Any way to create a URL from a byte array?](https://stackoverflow.com/questions/17776884/any-way-to-create-a-url-from-a-byte-array)
- [Writing a Protocol Handler](https://www.oreilly.com/library/view/learning-java/1565927184/apas02.html)
