## Facade Design Pattern
**Why?**
### Facade reduces coupeling of client and multiple interfaces and classes that it uses.
- Client has to interact with a large number of interfaces and classes in a subsystem to get result. So client gets tightly coupled with those interfaces & classes. Façade solves this problem.
- Façade provides a simple and unified interface to a subsystem. Client interacts with just the façade now to get same result.
- Façade is NOT just a one to one method forwarding to other classes.

### What is an Facade?
- A single service to communicate with multiple sources.

![UML](/Files/FacadeDP.png)


### Implement a Facade
- We start by creating a class that will serve as a facade
- We determine the overall "use cases"/tasks that the subsystem is used for.
- We write a method that exposes each "use case" or task.
- This method takes care of working with different classes of subsystem.

### Implementation Considerations
- A façade should minimize the complexity of subsystem and provide usable interface.
- You can have an interface or abstract class for façade and client can use different subclasses to talk to different subsystem implementations.
- A façade is not replacement for regular usage of classes in subsystem. Those can be still used outside of façade. Your subsystem class implementations should not make assumptions of usage of façade by client code.

### Design Considerations
- Façade is a great solution to simplify dependencies. It allows you to have a weak coupling between subsystems.
- If your only concern is coupling of client code to subsystem specific classes and not worried about simplification provided by a façade, then you can use abstract factory pattern in place of façade.

### Facade vs Adapter DP

Façade

- Intent is to simplify the usage of subsystem for client code.
- Façade is not restricted by any existing interface. It often defines simple methods which handle complex interactions behind scenes

Adapter

- Adapter is meant to simply adapt an object to different interface.
- Adapter is always written to confirm to a particular interface expected by client code. It has to implement all the methods from interface and adapt them using existing object.

### Pitfalls
- Not a pitfall of the pattern itself but needing a façade in a new design should warrant another look at API design.
- It is often overused or misused pattern & can hide improperly designed API. A common misuse is to use them as "containers of related methods". So be on the lookout for such cases during code reviews.

### Examples:
- Email Facade

![UML](/Files/EmailFacade.png)

**Eamil class**
```java
package com.coffeepoweredcrew.facade.email;

public class Email {

	public static EmailBuilder getBuilder() {
		return new EmailBuilder();
	}
}
```
**Email builder**
```java
package com.coffeepoweredcrew.facade.email;

public class EmailBuilder {

	public EmailBuilder withTemplate(Template template) {
		return this;
	}
	
	public EmailBuilder withStationary(Stationary stationary) {
		return this;
	}
	
	public EmailBuilder forObject(Object object) {
		return this;
	}
	
	public Email build() {
		return null;
	}
	
	public Email getEmail() {
		return null;
	}
}

```
**Stationary class**
```java
package com.coffeepoweredcrew.facade.email;

public interface Stationary {

	String getHeader();
	
	String getFooter();
}

```
**Stationary Factory**
```java
package com.coffeepoweredcrew.facade.email;

public class StationaryFactory {

	public static Stationary createStationary() {
		return new HalloweenStationary();
	}
}

```
**Template class**
```java
package com.coffeepoweredcrew.facade.email;

public abstract class Template {

	public enum TemplateType {Email, NewsLetter};
	
	public abstract String format(Object obj);
	
}
```
**Template Factory**
```java
package com.coffeepoweredcrew.facade.email;

import com.coffeepoweredcrew.facade.email.Template.TemplateType;

public class TemplateFactory {

	public static Template createTemplateFor(TemplateType type) {
		switch (type) {
		case Email:
			return new OrderEmailTemplate();
		default:
			throw new IllegalArgumentException("Unknown TemplateType");
		}
		
	}
}
```
**Halloween Stationary**
```java
package com.coffeepoweredcrew.facade.email;

public class HalloweenStationary implements Stationary {

	@Override
	public String getHeader() {
		return "It's Halloween!!";
	}

	@Override
	public String getFooter() {
		return "BUY MORE STUFF! It's Halloween, c'mon!!";
	}

}
```
**Mailer class**
```java
package com.coffeepoweredcrew.facade.email;

public class Mailer {

	private static final Mailer MAILER = new Mailer();
	
	public static Mailer getMailer() {
		return MAILER;
	}
	
	private Mailer() {
		
	}
	
	public boolean send(Email email) {
		return true;
	}; 
	
}
```
**Order Email Template**
```java
package com.coffeepoweredcrew.facade.email;

public class OrderEmailTemplate extends Template {

	@Override
	public String format(Object obj) {
		return "TEMPLATE";
	}
	
}
```
**Email Facade**
```java
package com.coffeepoweredcrew.facade.email;

import com.coffeepoweredcrew.facade.Order;
import com.coffeepoweredcrew.facade.email.Template.TemplateType;

//Facade provides simple methods for client to use
public class EmailFacade {
	
	public boolean sendOrderEmail(Order order) {
		Template template = TemplateFactory.createTemplateFor(TemplateType.Email);
		Stationary stationary = StationaryFactory.createStationary();
		Email email = Email.getBuilder()
					  .withTemplate(template)
					  .withStationary(stationary)
					  .forObject(order)
					  .build();
		Mailer mailer = Mailer.getMailer();
		return mailer.send(email);
	}
}
```
**Order class**
```java
package com.coffeepoweredcrew.facade;

public class Order {

	private String id;
	
	private double total;

	public Order(String id, double total) {
		this.id = id;
		this.total =total;
	}
	
	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public double getTotal() {
		return total;
	}

	public void setTotal(double total) {
		this.total = total;
	}
	
}
```
**Client Main**
```java
public class Client {

	public static void main(String[] args) {
		Order order = new Order("101", 99.99);
		EmailFacade facade = new EmailFacade();
		
		boolean result = facade.sendOrderEmail(order);
		
		System.out.println("Order Email "+ (result?"sent!":"NOT sent..."));
		
	}

    // Without Facade
	private static boolean sendOrderEmailWithoutFacade(Order order) {
		Template template = TemplateFactory.createTemplateFor(TemplateType.Email);
		Stationary stationary = StationaryFactory.createStationary();
		Email email = Email.getBuilder()
					  .withTemplate(template)
					  .withStationary(stationary)
					  .forObject(order)
					  .build();
		Mailer mailer = Mailer.getMailer();
		return mailer.send(email);
	}
	
}
```

### Existing examples
- java.net.URL
    - The java.net.URL class is a great example of façade. This class provides a simple method called as openStream() and we get an input stream to the resource pointed at by the URL object.
    - This class takes care of dealing with multiple classes from the java.net package as well as some internal sun packages.

![IOExample](/Files/FacadeDPExmple.png)
