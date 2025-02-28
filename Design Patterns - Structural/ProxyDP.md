## Proxy Design Pattern
**Why?**
### Placeholder to another object
- We need to provide a placeholder or surrogate to another object.
- Proxy acts on behalf of the object and is used for lots of reasons some of the main reasons are:
    - Protection Proxy - Control access to original object's operations
    - Remote Proxy - Provides a local representation of a remote object
    - Virtual proxy - Delays construction of original object until absolutely necessary
- Client is unaware of existence of proxy. Proxy performs its work transparently.
- Lazy loading of collections by hibernate, APO based method level security, RMI/Web service stubs are examples of real life proxy usage.

### What is an Proxy?
- Interface implementer by RealObject and a ProxyObject that have a reference to RealObject.
- Not all Proxy need a reference to RealObject!

![UML](/Files/ProxyDP.png)

### Implement a Proxy
- We start by implementing proxy
    - Proxy must implement same interface as the real subject
    - We can either create actual object later when required or ask for one in constructor.
    - In method implementations of proxy we implement proxy's functionality before delegating to real object
- How to provide client with proxies instance is decided by application. We can provide a factory or compose client code with proxies instance.
- What we are implementing above is also called as static proxy. Java also provides "dynamic proxies"

### Implement a Dynamic Proxy
- We start by implementing java.lang.reflect.InvocationHandler
    - Invocation handler implements invoke method which is called to handle every method invocation on proxy
    - We need to take action as per the method invoked. We'll cache the Method instances on image interface so that we can compare them inside invoke method
    - Our invocation handler will accept same argument in constructor as needed by constructor of real object
- Actual proxy instance is created using java.lang.reflect.Proxy by client.

### Implementation Considerations
- How proxy gets hold of the real object depends on what purpose proxy serves. For creation on demand type of proxies; actual object is created only when proxy can't handle client request. Authentication proxies use pre-built objects so they are provided with object during construction of proxy.
- Proxy itself can maintain/cache some state on behalf of real object in creation on demand use cases.
- Pay attention to performance cost of proxies as well synchronization issues added by proxy itself.

### Design Considerations
- Proxies typically do not need to know about the actual concrete implementation of real object.
- With Java you can use dynamic proxy allowing you to create proxies for any object at runtime.
- Proxies are great for implementing security or as stand-ins for real objects which may be a costly object that you want to defer loading. Proxies also make working with remote services/APIs easy by representing them as regular objects and possibly handling network communications behind the scene.

### Proxy vs Decorator DP
Proxy

- Depending on type of proxy it doesn't need real object all the time.
- Purpose of proxy is to provide features like access control, lazy loading, auditing etc.

Decorator

- A decorator needs to have a real object for it to work.
- A decorator is meant to add functionality to existing functionality provided by object & used by client directly.

### Pitfalls
- Java's dynamic proxy only works if your class is implementing one or more interfaces. Proxy is created by implementing these interfaces.
- If you need proxies for handling multiple responsibilities like auditing, authentication, as a stand-in for the same instance, then it's better to have a single proxy to handle all these requirements. Due to the way some proxies create object on their own, it becomes quite difficult to manage them.
- Static proxies look quite similar to other patterns like decorator & adapter patterns. It can be confusing to figure it out from code alone for someone not familiar with all these patterns.

### Examples:
- Postpone inplementation of real Object as long as possible

![UML](/Files/ProxyDPExample.png)

**Point2D Class**
```java
package com.coffeepoweredcrew.proxy;

public class Client {

	public static void main(String[] args) {
		Image img = ImageFactory.getImage("A1.bmp");
		
		img.setLocation(new Point2D(10,10));
		System.out.println("Image location :"+img.getLocation());
		System.out.println("rendering image now.....");
		img.render();
	}

}
```
**Image Interface**
```java
package com.coffeepoweredcrew.proxy;

//Interface implemented by proxy and concrete objects
public interface Image {

	void setLocation(Point2D point2d);
	Point2D getLocation();
	void render();

}
```
**Bitmap Image - A slow object creation as it loads from Disk**
```java
package com.coffeepoweredcrew.proxy;


//Our concrete class providing actual functionality
public class BitmapImage implements Image {
	
	private Point2D location;
	private String name;
	
	public BitmapImage(String filename) {
		//Loads image from file on disk
		System.out.println("Loaded from disk:"+filename);
		name = filename;
	}
	
	@Override
	public void setLocation(Point2D point2d) {
		location = point2d;
	}

	@Override
	public Point2D getLocation() {
		return location;
	}

	@Override
	public void render() {
		//renders to screen
		System.out.println("Rendered "+this.name);
	}
	
}
```
**Proxy Image**
```java
package com.coffeepoweredcrew.proxy;

//Proxy class.
public class ImageProxy implements Image {

	private String name;
	
	private BitmapImage image;
	
	private Point2D location;

	public ImageProxy(String name) {
		this.name = name;
	}
	
	@Override
	public void setLocation(Point2D point2d) {
		if(image != null) {
			image.setLocation(point2d);
		} else {
			location = point2d;
		}
	}

	@Override
	public Point2D getLocation() {
		if(image != null) {
			return image.getLocation();
		}
		return location;
	}

	@Override
	public void render() {
		if(image == null) {
			image = new BitmapImage(name);
			if(location != null) {
				image.setLocation(location);
			}
		}
		image.render();
	}
		
}
```
**Image Factory**
```java
package com.coffeepoweredcrew.proxy;

//Factory to get image objects. 
public class ImageFactory {
	//We'll provide proxy to caller instead of real object
	
	public static Image getImage(String name) {
		return new ImageProxy(name);
	}
}
```
**Client Main Class**
```java
package com.coffeepoweredcrew.proxy;


public class Client {

	public static void main(String[] args) {
		Image img = ImageFactory.getImage("A1.bmp");
		
		img.setLocation(new Point2D(10,10));
		System.out.println("Image location :"+img.getLocation());
		System.out.println("rendering image now.....");
		img.render();
	}

}
```
===
=

- Dynamic Proxy

**Image Invocation Handler**
```java
package com.coffeepoweredcrew.proxy.dynamic;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

import com.coffeepoweredcrew.proxy.BitmapImage;
import com.coffeepoweredcrew.proxy.Image;

import javafx.geometry.Point2D;

//Implement invocation handler. Your "proxy" code goes here.
public class ImageInvocationHandler implements InvocationHandler{

	private String filename;
	private Point2D location;
	private BitmapImage image;
	private static final Method setLocationMethod;
	private static final Method getLocationMethod;
	private static final Method renderMethod;
	//Cache Methods for later comparison 
	static {
		try {
			setLocationMethod = Image.class.getMethod("setLocation", new Class[]{Point2D.class});
			getLocationMethod = Image.class.getMethod("getLocation", null);
			renderMethod = Image.class.getMethod("render", null);
		} catch (NoSuchMethodException e) {
			throw new NoSuchMethodError(e.getMessage());
		}
	}

	public ImageInvocationHandler(String filename) {
		this.filename = filename;	
	}
	//This method is called for eery method invocation on the proxy
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		//You can implement proxy logic here 
		System.out.println("Invoking methood: "+method.getName());
		if(method.equals(setLocationMethod)) {
			return handleSetLocation(args);
		} else if(method.equals(getLocationMethod)) {
			return handleGetLocation();
		} else if(method.equals(renderMethod)) {
			return handleRender();
		}
		return null;
	}
	//We create real object only when we really need it
	private Object handleRender() {
		if(image == null) {
			image = new BitmapImage(filename);
			if(location != null) {
				image.setLocation(location);
			}
		}
		image.render();
		return null;
	}

	private Point2D handleGetLocation() {
		if(image != null) {
			return image.getLocation();
		} else {
			return this.location;
		}
	}

	private Object handleSetLocation(Object[] args) {
		if(image != null) {
			image.setLocation((Point2D)args[0]);
		} else {
			this.location = (Point2D)args[0];
		}
		return null;
	}
	
}
```
**Image Factory**
```java
package com.coffeepoweredcrew.proxy.dynamic;

import java.lang.reflect.Proxy;
import com.coffeepoweredcrew.proxy.Image;

//Factory to get image objects. 
public class ImageFactory {
	//We'll provide proxy to caller instead of real object
	
	public static Image getImage(String name) {
		return (Image)Proxy.newProxyInstance(ImageFactory.class.getClassLoader(), new Class[]{Image.class},
				new ImageInvocationHandler(name)); 
	}
}
```
**Client Main Class**
```java
package com.coffeepoweredcrew.proxy.dynamic;

import com.coffeepoweredcrew.proxy.Image;
import javafx.geometry.Point2D;

public class Client {

	public static void main(String[] args) {
		Image img = ImageFactory.getImage("A.bmp");
		img.setLocation(new Point2D(-10, 0));
		System.out.println(img.getLocation());
		System.out.println("*****************************");
		img.render();
		
	}
}
```

### Existing examples
- This is one pattern where you'll find numerous examples
- Hibernate uses a proxy to load collections of value types. If you have a relationship in entity class mapped as a collection, marked as candidate for lazy loading then Hibernate will provide a virtual proxy in its place.
- Spring uses proxy pattern to provide support for features like transactions, caching and general AOP support.
- Hibernate & spring both can create proxies for classes which do not implement any interface. They use third party frameworks like cglib, aspect to create dynamic proxies (remember, Java's dynamic proxy needs interface) at runtime.
