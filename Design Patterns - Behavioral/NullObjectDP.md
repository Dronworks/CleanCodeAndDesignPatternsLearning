## Null Object Pattern
**Why?**
### Alternate representation to indicate absence of object
- We use "null" value to represent an absence of object. Using "Null Object" pattern we can provide an alternate representation to indicate an absence of object.
- Most important characteristic of a null object is that it'll basically do nothing & store nothing when an operation is called on it.
- Null object seems like a proxy as it stands in for a real object, however a proxy at some point will use real object or transform to a real object & even in absence of the real object proxy will provide some behaviour with side effect. Null object will not do any such thing. Null objects don't transform into real objects.
- We use this pattern when we want to treat absence of a collaborator transparently without null checks.

### What is a Null Object?
- Use this when you want to handle a missing collaborator — meaning an object your class depends on — transparently, without adding null checks everywhere

![UML](/Files/NullObjectDPUML.png)


### Implement a Null Object
- We create a new class that represents our null object by extending from base class or implementing given interface.
- In the null object implementation, for each method we'll not do anything. **However doing nothing can mean
different things in different implementations.**  `E.g. If a method in a null object returns something then we can either return another null object, a predefined default value or null.`
- Code which creates objects of our implementation will create & pass our null object in a specific situation.

### Implementation Considerations
- Class which is using Null Object should not have to do anything special when working with this object.
- What "do nothing" means for an operation can be different in different classes. This is especially true where methods in null objects are expected to return values. `E.g. if methon returns int, decide what is null 0 or 1`
- If you find a need where your null object has to transform into a real object then you better use something like state pattern with a null object as one of the states.

### Design Considerations
- Since null objects don't have a state & no complex behavior they are good candidates for singleton pattern. We can use a single instance of null object everywhere.
- Null objects are useful in many other design patterns like state - to represent a null state, in strategy pattern to provide a strategy where no action is taken on input. `E.g. canceling order can be a null design pattern as we don't want to cancel the canceling order` and `E.g.2 in strategy design pattern we can have a null object as one of the strategies to return object as it is`

### Null Object vs Proxy DP

Null Object

- Null objects never transform/create or provide an indirection to real object.
- Null objects do not "act on behalf" of real object. Its job is to do nothing.

Proxy

- Many types of proxies will need a real object eventually.
- In absence of real object, proxies will provide behavior matching to real object.

### Pitfalls
- Creating a proper Null object may not be possible for all classes. Some classes may be expected to cause a change, and absence of that change may cause other class operations to fail. `E.g. if we "saving" an object to file system, and then need to send it in email. The email will fail`
- Finding what "do nothing" means may not be easy or possible. If our null object method is expected to return another object then this problem is more apparent.

### Examples:
- Save report to disk with option not to do anythin

![UML](/Files/NullObjectDPExampleUml.png)

**ComplexService class**
```java
public class ComplexService {
    private Storageservice storage;
    private String reportName;
    
    public ComplexService(StorageService storage) {
        this.storage = storage;
        reportName = "A Complex Report";
    }

    public ComplexService (String reportName, StorageService storage) {
        this.storage = storage;
        this.reportName = reportName;
    }

    public void generateReport () {
        System.out.println("Starting a complex report build!");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Done with report..");
        storage.save(new Report(reportName));
    }

}
```
**Storage service class**
```java
public class StorageService {
    public void save(Report report) {
        System.out.println("Writing report out");
        try(PrintWriter writer = new PrintWriter(report.getName() + ".txt")) {
            writer.println(report.getName());
        } catch (IOException e) {
            e. printStackTrace () ;
        }
    }
}
```
**NullStorage service class** - overrides **ALL** methods
```java
public class NullStorageService extends StorageService {
    @Override
    public void save(Report report) {
        System.out.println("Null object save method. Doing nothing");
    }
}
```
**Report class**
```java
public class Report {
    private String name;

    public Report(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

### Existing examples
- The various adapter classes from java.awt.event package can be thought of as examples null object. Only reason they are not the examples of this pattern is that they are abstract classes but without any abstract method.

```java
public abstract class MouseAdapter implements MouseLis
    /**
    * {@inheritDoc}
    */
    public void mouseClicked(MouseEvent e) {}
    /**
    * {@inheritDoc}
    */
    public void mousePressed(MouseEvent e) {}
    /**
    * {@inheritDoc}
    */
    public void mouseReleased(MouseEvent e) {}
    /**
    * {@inheritDoc}
    */
    public void mouseEntered(MouseEvent e) {}
    /**
    * {@inheritDoc}
    */
    public void mouseExited(MouseEvent e) {}
    ...
}
```