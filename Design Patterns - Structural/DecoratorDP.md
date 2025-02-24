## Decorator Design Pattern
**Why?**
### Enhance behaviour of an Existing object
- When we want to enhance behaviour of our existing object dynamically as and when required then we can use decorator design pattern.
- Decorator wraps an object within itself and provides same interface as the wrapped object. So the client of original object doesn't need to change.
- A decorator provides alternative to subclassing for extending functionality of existing classes.
- Open/Closed Principle - instead of extending already created service with new fuctionality, just extend with decorator.

### What is an Decorator?
- A class that implements an interface and holds an instance of this interface.

![UML](/Files/DecoratorDP.png)


### Implement a Decorator
- A Component that defines interface needed or already used by client.
- Concrete component implementation.
- Decorator that implements component & also needs reference to concrete component.
- Decorator methods provide additional behaviour on top that provided by concrete component instance.
- **A decorator can be abstract.

### Implementation Considerations
- Since we have decorators and concrete classes extending from common component, avoid large state in this base class as decorators may not need all that state.
- Pay attention to equals and hashCode methods of decorator. When using decorators, you have to decide if decorated object is equal to same instance without decorator.
- Decorators support recursive composition, and so this pattern lends itself to creation of lots of small objects that add "just a little bit" functionality. Code using these objects becomes difficult to debug.

### Design Considerations
- Decorators are more flexible & powerful than inheritance. Inheritance is static by definition but decorators allow you to dynamically compose behaviour using objects at runtime.
- Decorators should act like additional skin over your object. They should add helpful small behaviours to object's original behaviour. Do not change meaning of operations.

### Decorator vs Composite DP

Decorator

- Intent is to "add to" existing behaviour of existing object.
- Decorator can be thought as degenerate composite with only one component.

Composite

- Composites are meant for object aggregation only.
- Composites support any number of components in aggregation.

### Pitfalls
- Often results in large number of classes being added to system, where each class adds a small amount of functionality. You often end up with lots of objects, one nested inside another and so on.
- Sometimes new comers will start using it as a replacement of inheritance in every scenario. Think of decorators as a thin skin over existing object.

### Examples:
- Docorator that convert a message to different format

![UML](/Files/DecoratorDPExample.png)

**Message interface**
```java
//Base interface or component
public interface Message {
    String getContent();
}
```
**Message Implementation**
```java
//Concrete component. Object to be decorated
public class TextMessage implements Message {

    private String msg;

    public TextMessage(String msg) {
        this.msg = msg;
    }

    @Override
    public String getContent() {
        return msg;
    }
}
```
**Main client**
```java
public class Client {

    public static void main(String[] args) {
        Message m = new TextMessage("The <FORCE> is strong with this one!");
        System.out.println(m.getContent());

        Message decorator = new HtmlEncodedDecorator(m);
        System.out.println(decorator.getContent());
        
        decorator = new Base64EncodedMessage(decorator); // Not must but will do a recoursion decoration
        System.out.println(decorator.getContent());
    }

}
```
**HTML Decorator**
```java
import org.apache.commons. text.StringEscapeUtils;

//Decorator. Implements component interface
public class HtmlEncodedMessage implements Message {

    private Message msg;

    public HtmlEncodedMessage(Message msg) {
        this.msg = msg;
    }

    @Override
    public String getContent() {
        return StringEscapeUtils.escapeHtml4(msg.getContent());
    }
}
```
**Base64 encoder decorator**
```java
import java.util.Base64;

public class Base64EncodedMessage implements Message {

    private Message msg;

    public Base64EncodedMessage(Message msg) {
        this.msg = msg;
    }

    @Override
    public String getContent() {
        //Be wary of charset !! This is platform dependent ..
        return Base64.getEncoder().encodeToString(msg.getContent().getBytes());
    }

}
```
- Dynamic Car composition
```java
public class Car {
    public void drive() {
        System.out.println("Driving the car...");
    }
}
```
```java
public class AirConditioningDecorator extends Car {
    private Car car;

    public AirConditioningDecorator(Car car) {
        this.car = car;
    }

    @Override
    public void drive() {
        car.drive();  // Call the original drive method
        System.out.println("Air conditioning is on.");
    }
}
```
```java
public class BluetoothDecorator extends Car {
    private Car car;

    public BluetoothDecorator(Car car) {
        this.car = car;
    }

    @Override
    public void drive() {
        car.drive();  // Call the original drive method
        System.out.println("Bluetooth is connected.");
    }
}
```
```java
public class NavigationDecorator extends Car {
    private Car car;

    public NavigationDecorator(Car car) {
        this.car = car;
    }

    @Override
    public void drive() {
        car.drive();  // Call the original drive method
        System.out.println("Navigation system is active.");
    }
}
```
```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        // Base car
        Car car = new Car();

        // Simulating user preferences or configuration to apply decorators
        List<String> features = new ArrayList<>();
        features.add("AirConditioning");  // Example: user wants Air Conditioning
        features.add("Bluetooth");        // Example: user wants Bluetooth

        // Dynamically apply decorators based on the preferences
        for (String feature : features) {
            if (feature.equals("AirConditioning")) {
                car = new AirConditioningDecorator(car);
            }
            if (feature.equals("Bluetooth")) {
                car = new BluetoothDecorator(car);
            }
            if (feature.equals("Navigation")) {
                car = new NavigationDecorator(car);
            }
        }

        // Now, drive the car with the applied features
        car.drive();
    }
}
```
### Existing examples
- I/O package. E.g java.io.BufferedOutputStream that decorates the java.io.OutputStream (improves the disk i/o prerformance by reducing number of writes)

![IOExample](/Files/DecoratorDPIOExample.png)
