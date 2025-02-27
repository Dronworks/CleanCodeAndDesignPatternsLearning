## Flyweight Design Pattern
**Why?**
### A need of a large number of object that are performance consern to maintain
- Our system needs a large number of objects of a particular class & maintaining these instances is a performance concern.
- Flyweight allows us to share an object in multiple contexts. But instead of sharing entire object, which may not be feasible, we divide object state in two parts: intrinsic (state that is shared in every context) & extrinsic (context specific state). We create objects with only intrinsic state and share them in multiple contexts.
- Client or user of object provides the extrinsic state to object to carry out its functionality.
- We provide a factory so client can get required flyweight objects based on some key to identify flyweight.

### What is a Flyweight?

![UML](/Files/FlyweightDP.png)

### Implement a xxx
- We start by identifying "intrinsic" & "extrinsic" state of our object
- We create an interface for flyweight to provide common methods that accept extrinsic state
- In implementation of shared flyweight we add intrinsic state & also implement methods.
- In unshared flyweight implementation we simply ignore the extrinsic state argument as we have all state within object.
- Next we implement the flyweight factory which caches flyweights & also provides method to get them
- In our client we either maintain the extrinsic state or compute it on the fly when using flyweight

### Implementation Considerations
- A factory is necessary with flyweight design pattern as client code needs easy way to get hold of shared flyweight. Also number of shared instances can be large so a central place is good strategy to keep track of all of them.
- Flyweight's intrinsic state should be immutable for successful use of flyweight pattern.

### Design Considerations
- Usability of flyweight is entirely dependent upon presence of sensible extrinsic state in object which can be moved out of object without any issue.
- Some other design patterns like state and strategy can make best use of flyweight pattern.

### Flyweight vs Object Pool DP

Flyweight

- State of flyweight object is divided. Client must provide part of state to it.
- In a typical usage client will not change intrinsic state of flyweight instance as it is shared.

Object Pool

- A pooled object contains all of its state encapsulated within itself.
- Clients can and will change state of pooled objects.

### Pitfalls
- Runtime cost may be added for maintaining extrinsic state. Client code has to either maintain it or compute it every time it needs to use flyweight.
- It is often difficult to find perfect candidate objects for flyweight. Graphical applications benefit heavily from this pattern however a typical web application may not have a lot of use for this pattern.

### Examples:
- ErrorMessage + ErrorBannedMessage

![UML](/Files/FlyweightDPExample.png)

**ErrorMessage**
```java
package com.coffeepoweredcrew.flyweight;

// Interface implemented by Flyweights
public interface ErrorMessage {
	// Get error message
	String getText(String code);
}
```
**SystemErrorMessage**
```java
package com.coffeepoweredcrew.flyweight;

// A concrete Flyweight. Instance is shared
public class SystemErrorMessage implements ErrorMessage{
	
	// Some error message $errorCode
	private String messageTemplate;
	
	// http://somedomain.com/help?error=
	private String helpUrlBase;
	
	public SystemErrorMessage(String messageTemplate, String helpUrlBase) {
		this.messageTemplate = messageTemplate;
		this.helpUrlBase = helpUrlBase;
	}
	
	@Override
	public String getText(String code) {
		return messageTemplate.replace("$errorCode", code) + helpUrlBase+code;
	}

}
```
**UserBannedErrorMessage**
```java
package com.coffeepoweredcrew.flyweight;

import java.time.Duration;

// Unshared concrete flyweight.
public class UserBannedErrorMessage implements ErrorMessage {
    // All state is defined here
    private String caseId;

    private String remarks;

    private Duration banDuration;

    private String msg;

    public UserBannedErrorMessage(String caseId) {
        // Load case info from DB.
        this.caseId = caseId;
        remarks = "You violated terms of use.";
        banDuration = Duration.ofDays(2);
        msg = "You are BANNED. Sorry. \nMore information:\n";
        msg += caseId +"\n";
        msg += remarks+"\n";
        msg += "Banned For:" +banDuration.toHours() + " Hours";
    }
    // We ignore the extrinsic state argument
    @Override
    public String getText(String code) {
        return msg;
    }

    public String getCaseNo() {
        return caseId;
    }

}
```
**ErrorMessage Factory**
```java
package com.coffeepoweredcrew.flyweight;

import java.util.HashMap;
import java.util.Map;

// Flyweight factory. Returns shared flyweight based on key
public class ErrorMessageFactory {
	
	// This serves as key for getting flyweight instance
	public enum ErrorType {GenericSystemError, PageNotFoundError, ServerError}
	
	private static final ErrorMessageFactory FACTORY = new ErrorMessageFactory();

	public static ErrorMessageFactory getInstance() {
		return FACTORY;
	}
	
	private Map<ErrorType, SystemErrorMessage> errorMessages = new HashMap<>();
	
	private ErrorMessageFactory() {
		errorMessages.put(ErrorType.GenericSystemError, 
				new SystemErrorMessage("A genetic error of type $errorCode occured. Please refer to:\n",
						"http://google.com/q="));
		errorMessages.put(ErrorType.PageNotFoundError, 
				new SystemErrorMessage("Page not foun. An error of type $errorCode occured. Please refer to:\n",
						"http://google.com/q="));
	}
	
	public SystemErrorMessage getError(ErrorType type) {
		return errorMessages.get(type);
	}
	
	public UserBannedErrorMessage getUserBannedMessage(String caseId) {
		return new UserBannedErrorMessage(caseId);
	}
}
```
**Client Main**
```java
package com.coffeepoweredcrew.flyweight;

import com.coffeepoweredcrew.flyweight.ErrorMessageFactory.ErrorType;

public class Client {

	public static void main(String[] args) {
		
		SystemErrorMessage msg1 = ErrorMessageFactory.getInstance().getError(ErrorType.GenericSystemError);
		System.out.println(msg1.getText("4056"));
		
		UserBannedErrorMessage msg2 = ErrorMessageFactory.getInstance().getUserBannedMessage("1202");
		System.out.println(msg2.getText(null));
	}

}
```

### Existing examples
- Java uses flyweight pattern for Wrapper classes like java.lang.Integer, Short, Byte etc. Here the valueOf static method serves as the factory method.
    - All the integer range already created, and we just get it via valueOf.

![IOExample](/Files/FlyweightExternalExample.png)

- String pool which is maintained by JVM is also an example of flyweight. We can call the intern() method on a String object to explicitly request this String object to be interned. This method will returned a reference to already cached object if present or else will create new String in cache if not present.
