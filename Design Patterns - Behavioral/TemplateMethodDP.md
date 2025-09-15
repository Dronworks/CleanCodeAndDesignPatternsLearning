## Template Method Design Pattern
**Why?**
### Define algorithem in a method as a series of steps
- Each step is a method call, and each such method is an abstract in this class
- Subclasses will extend this class and provide implementations


### What is a Template Method?
- A way to store algorithm steps with abstract steps implementation, but with the algorithm order.
- It is an example of **inversion of control** don't call us we will call you!

![UML](/Files/TemplateMethodDPUML.png)


### Implement a Template Method
- Define algorithm in a method with multiple steps and abstract method
- Don't create too small steps and too many methods, but give enough freedom for implementations
- Implements abstract methods

### Implementation Considerations
- A balance should be kept on how many steps are we split to. If too many it is overhead, if too low we don't give enough control to subclass
- Mark the algorithm as final (if needed) so implementations won't change it

### Design Considerations
- We can use inheritance subclasses (to override only what we need)
- Factory method DP often used in TemplateDP

### Template Method vs Strategy

Template Method

- All subclasses implement the steps for the exact same algorithm.
- Client code relies solely on inheritance to get a variation of same algorithm.

Strategy

- In strategy design pattern each subclasses represents a separate algorithm.
- Client code uses composition principal to configure main class with chosen strategy object.

### Pitfalls
- Tracking down what code executed as part of our algorithm requires looking up multiple classes. The
problem becomes more apparent if subclasses themselves start using inheritance themselves to reuse
only some of the existing steps & customize a few.
- Unit testing can become a little more difficult as the individual steps may require some specific state values to be present.

### Examples:
- Print with different algorithm implementation

![UML](/Files/TemplateMethodDPExample.png)

**Order class**
```java
package com.coffeepoweredcrew.strategy;

import java.time.LocalDate;
import java.util.HashMap;
import java.util.Map;

public class Order {

    private String id;

    private LocalDate date;

    private Map<String, Double> items = new HashMap<>();

    private double total;

    public Order(String id) {
        this.id = id;
        date = LocalDate.now();
    }

    public String getId() {
        return id;
    }

    public LocalDate getDate() {
        return date;
    }

    public Map<String, Double> getItems() {
        return items;
    }

    public void addItem(String name, double price) {
        items.put(name, price);
        total+= price;
    }

    public double getTotal() {
        return total;
    }

    public void setTotal(double total) {
        this.total = total;
    }
}
```
**OrderPrinter class - template**
```java
//Abstract base class defines the template method
public abstract class OrderPrinter {

    public void printOrder (Order order, String filename) throws IOException {
        try (PrintWriter writer = new PrintWriter(filename)){
            writer.println(start());

            writer.println(formatOrderNumber(order));

            writer.println(formatItems(order));

            writer.println(formatTotal(order));

            writer.println(end());
        }
    }

    protected abstract String start();

    protected abstract String formatOrderNumber(Order order);

    protected abstract String formatItems(Order order);

    protected abstract String formatTotal(Order order);

    protected abstract String end();
}
```
**Text Printer - extends and implements for text**
```java
public class TextPrinter extends OrderPrinter{

    @Override
    protected String start() {
        return "Order Details";
    }

    @Override
    protected String formatOrderNumber (Order order) {
        return "Order #"+order.getId();
    }

    @Override
    protected String formatItems(Order order) {
        StringBuilder builder = new StringBuilder("Items\n------\n");

        for (Map. Entry<String, Double> entry : order.getItems() .entrySet()) {
            builder.append(entry.getKey()+ " $"+entry.getValue() + "\n");
        }
        builder.append("------------");
        return null;
    }

    @Override
    protected String formatTotal(Order order) {
        return "Total: $"+order.getTotal();
    }

    @Override
    protected String end() {
        return "";
    }

}
```
**Html Printer - extends and implements for html**
```java
public class HtmlPrinter extends OrderPrinter {

    @Override
    protected String start() {
        return "<html><head><title>Order Details</title></head><body>";
    }

    @Override
    protected String formatOrderNumber (Order order) {
        return "<h1>Order #"+order.getId()+"</h1>";
    }

    @Override
    protected String formatItems(Order order) {
        StringBuilder builder = new StringBuilder("<p><ul>");
        for (Map. Entry<String, Double> e : order.getItems().entrySet()) {
            builder.append("<li>"+e.getKey()+" $"+e.getValue()+"</li>");
        }
        builder.append("</ul></p>");
        return builder.toString();
    }

    @Override
    protected String formatTotal(Order order) {
        return "<br/><hr/><h3>Total : $"+order.getTotal()+"</h3>";
    }

    @Override
    protected String end() {
        return "</body></html>";
    }
}
```
**Client Main Class**
```java
public class Client {

    public static void main(String[] args) throws IOException {
        Order order = new Order("1001");

        order.addItem("Soda", 2.50);
        order.addItem("Sandwitch", 11.95);
        order.addItem("Pizza", 15.95);

        OrderPrinter printer = new TextPrinter();
        printer.printOrder(order, "1001. txt") ;
    }
}
```

### Existing examples
- AbstractMap or AbstractSet are classes that are examples, where size needs to be calculated.

    ![EXAMPLE](/Files/AbstractSetExample.png)