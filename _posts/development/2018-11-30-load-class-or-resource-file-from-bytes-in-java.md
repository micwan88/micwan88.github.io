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


If the resource file cannot be found by its parent, it will try load the resource file by `findResource()` itself. If it is failed again, return `null` directly.

So if we want to implement a classloader to use bytes array for resource file, we should override the method of `findResource()`. Actually we can override `getResourceAsStream()` instead, but it will lose the hierarchy loading feature (i.e. you cannot get the resource file which is in parent classloader).


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


By using the above method, you can load normal / other class from parent classloader as well (i.e. without lose its hierarchy loading feature).

For this, I have made a helper class `ByteClassLoader` in which we can load class from bytes array no matter it is a single .class or a .jar file. It is open source and you can get it from here [helperclass4j](https://github.com/micwan88/helperclass4j).


### For Resource File - override findResource() to use bytes array

To override `findResource()` for using bytes array is a bit tricky as this method is return an `URL` class, so we got two more classes to be extended for making URL class by bytes array. The classes to be extended are are `URLStreamHandler` and `URLConnection`.

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
	protected URL findResource(String paramString) {
		byte[] extractedBytes = byteDataMap.get(paramString);
		if (extractedBytes != null) {
			try {
				return new URL(null, "bytes:///" + paramString, new Handler(extractedBytes, paramString));
			} catch (MalformedURLException e) {
				//Do nothing
			}
		}
		return null;
	}
}

//Define Custom URLStreamHandler
public class Handler extends URLStreamHandler {
	private byte[] byteContent = null;
	private String resourceName = null;

	/**
	 * @param byteContent
	 * @param resourceName
	 */
	public Handler(byte[] byteContent, String resourceName) {
		this.byteContent = byteContent;
		this.resourceName = resourceName;
	}
	
	public void setByteContent(byte[] byteContent, String resourceName) {
		this.byteContent = byteContent;
		this.resourceName = resourceName;
	}

	@Override
	protected URLConnection openConnection(URL paramURL) throws IOException {
		if (byteContent == null || resourceName == null)
			throw new UnsupportedOperationException("This handler only support to be created with byte array in constructor");
		
		//Resource not match
		if (!paramURL.getFile().endsWith(resourceName))
			throw new UnsupportedOperationException("URL file (" + paramURL.getFile() + ") name does not match with assigned resource name: " + resourceName);
		
		ByteURLConnection byteURLConnection = new ByteURLConnection(paramURL, byteContent);
		
		return byteURLConnection;
	}

}

//Define Custom ByteURLConnection
public class ByteURLConnection extends URLConnection {
	private byte[] byteContent = null;
	private ByteArrayInputStream byteInStream = null;
	
	protected ByteURLConnection(URL paramURL) {
		super(paramURL);
	}
	
	/**
	 * @param paramURL
	 * @param byteContent
	 */
	public ByteURLConnection(URL paramURL, byte[] byteContent) {
		super(paramURL);
		this.byteContent = byteContent;
	}
	
	@Override
	public InputStream getInputStream() throws IOException {
		if (byteInStream == null)
			connect();
		
		return byteInStream;
	}

	@Override
	public void connect() throws IOException {
		if (byteContent == null)
			throw new IOException("This handler only support to be created with byte array in constructor");
		
		byteInStream = new ByteArrayInputStream(byteContent);
	}
}

//Example Usage
public static void main(String[] args) throws IOException {
	//prepare the bytes array
	byte[] byteData = .....;

	ByteClassLoader byteClassLoader = new ByteClassLoader(this.getClass().getClassLoader());
	//Load bytes into hashmap
	byteClassLoader.loadDataInBytes(byteData, "some.properties");

	InputStream inputStream = byteClassLoader.getResourceAsStream("some.properties");
}
```


By using the above method, you can load other resource file from parent classloader as well (i.e. without lose its hierarchy loading feature).

Same as Java class, there is a open source helper class `ByteClassLoader` in which we can load resource file from bytes array no matter it is a single file or resources from .jar file. Please refer to [helperclass4j](https://github.com/micwan88/helperclass4j) for `ByteClassLoader`.


References:
- [Class Loaders in Java](https://www.baeldung.com/java-classloaders)
- [Any way to create a URL from a byte array?](https://stackoverflow.com/questions/17776884/any-way-to-create-a-url-from-a-byte-array)
- [Writing a Protocol Handler](https://www.oreilly.com/library/view/learning-java/1565927184/apas02.html)
