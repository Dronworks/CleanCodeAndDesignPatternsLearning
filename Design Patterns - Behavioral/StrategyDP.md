## Strategy Design Pattern
**Why?**
### Store algorithms in a separate class and make them interchangeable.
- Strategy pattern allows us to encapsulate an algorithm in a class. So now we can configure our context or main object with an object of this class, to change the algorithm used to perform given operation.
- This is really helpful if you have many possible variations of an algorithm.
- A good indication for applicability of strategy pattern is if we find different algorithms/behaviors in our methods which are selected with conditional statements like if-else or switch-case.
- Strategy classes are usually implemented in an inheritance hierarchy so that we can choose any one implementation and it'll work with our main object/context as the interface is same for all implementations.

### What is a Strategy?
- A way to define an algorithm in a separate class and make them interchangeable.

![UML](/Files/StrategyDP.png)


### Implement a Strategy
- We start by defining strategy interface which is used by our main/context class. Context class provides strategy with all the data that it needs.
- We provide implementations for various algorithms by implementing strategy interface a class per algorithm.
- Our context class provides a way to configure it with one of the strategy implementations. Client code will create context with one of the strategy object.

### Implementation Considerations
- We can implement our context in a way where strategy object is optional. This makes context usable for client codes who do not want to deal with concrete strategy objects.
- Strategy objects should be given all data they need as arguments to its method. If number of arguments are high then we can pass strategy an interface reference which it queries for data. Context object can implement this interface and pass itself to strategy.
- Strategies typically end up being stateless objects making them perfect candidates for sharing between context objects.

### Design Considerations
- Strategy implementations can make use of inheritance to factor out common parts of algorithms in base classes making child implementations simpler.
 - Since strategy objects often end up with no state of their own, we can use flyweight pattern to share them between multiple context objects.

### Strategy vs State DP

Strategy

- We create a class per algorithm.
- Strategy objects do not need to know about each other.

State

- In state pattern we have a class per state.
- If states are responsible for triggering state transitions then they have to know about at least next state.

### Pitfalls
- Since client code configures context object with appropriate strategy object, clients know about all implementations of strategy. Introducing new algorithm means changing client code as well.

### Examples:
- Print with different algorithms

![UML](/Files/PrintAlgorithmStrategy.png)

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
**OrderPrinter class - strategy**
```java
package com.coffeepoweredcrew.strategy;

import java.util.Collection;

// Strategy
public interface OrderPrinter {
	
	void print(Collection<Order> orders);
}
```
**Print service - context**
```java
package com.coffeepoweredcrew.strategy;


import java.util.LinkedList;

// Context 
public class PrintService {

	private OrderPrinter printer;
	
    public PrintService(OrderPrinter printer) {
    	this.printer = printer;
    }

    public void printOrders(LinkedList<Order> orders) {
        printer.print(orders);
    }
}
```
**Summary Printer**
```java
package com.coffeepoweredcrew.strategy;

import java.util.Collection;
import java.util.Iterator;

//Concrete strategy
public class SummaryPrinter implements OrderPrinter{

	@Override
	public void print(Collection<Order> orders) {
		System.out.println("*************** Summary Report *************");
		Iterator<Order> iterator = orders.iterator();
		double total = 0;
		for(int i=1; iterator.hasNext(); i++) {
			Order order = iterator.next();
			System.out.println(i +". "+order.getId()+ "\t"+order.getDate()+"\t"+order.getItems().size()+"\t"+order.getTotal());
			total += order.getTotal();
		}
		System.out.println("*******************************************");
		System.out.println("\t\t\t  Total "+total);
	}
	
}
```
**Details Printer**
```java
package com.coffeepoweredcrew.strategy;

import java.util.Collection;
import java.util.Iterator;
import java.util.Map;

public class DetailPrinter implements OrderPrinter {

    @Override
    public void print(Collection<Order> orders) {
        System.out.println("************* Detail Report ***********");
        Iterator<Order> iter = orders.iterator();
        double total = 0;
        for(int i=1;iter.hasNext();i++) {
            double orderTotal = 0;
            Order order = iter.next();
            System.out.println(i+". "+order.getId()+" \t"+order.getDate());
            for(Map.Entry<String, Double> entry : order.getItems().entrySet()) {
                System.out.println("\t\t"+entry.getKey()+"\t"+entry.getValue());
                orderTotal+=entry.getValue();
            }
            System.out.println("----------------------------------------");
            System.out.println("\t\t Total  "+orderTotal);
            System.out.println("----------------------------------------");
            total += orderTotal;
        }
        System.out.println("----------------------------------------");
        System.out.println("\tGrand Total "+total);
    }
}
```
**User**
```java
package com.coffeepoweredcrew.strategy;

public class User {
}
```
**Client Main Class**
```java
package com.coffeepoweredcrew.strategy;

import java.util.LinkedList;

public class Client {

    private static LinkedList<Order> orders = new LinkedList<>();

    public static void main(String[] args) {
        createOrders();
        //print all orders
        PrintService service = new PrintService(new DetailPrinter());
        service.printOrders(orders);
        
    }

    private static void createOrders() {
        Order o = new Order("100");
        o.addItem("Soda", 2);
        o.addItem("Chips", 10);
        orders.add(o);

        o = new Order("200");
        o.addItem("Cake", 20);
        o.addItem("Cookies", 5);
        orders.add(o);

        o = new Order("300");
        o.addItem("Burger", 8);
        o.addItem("Fries", 5);
        orders.add(o);
    }
}
```



### Existing examples
- The java.util.Comparator is a great example of strategy pattern. We can create multiple implementations of comparator, each using a different algorithm to perform comparison and supply those to various sort methods.

```java
class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```
```java
class SortByAge implements Comparator<User> {
    @Override
    public int compare(User ol, User o2) {
        return ol.getAge() - o2.getAge();
    }
}
```
```java
class SortByName implements Comparator<User> {
    @Override
    public int compare(User o1, User o2) {
        return o1.getName().compareToIgnoreCase(o2.getName());
    }
}
```
```java
List<User> list = new ArrayList<>();
list.add(new User("Nancy", 16));
list.add(new User("Dustin", 12));
list.add(new User("Steve", 17));
list.add(new User("Mike", 12));
list.add(new User("Max", 13));

list.sort(new SortByAge());

list.sort(new SortByName());
```

- Another example of strategy pattern is the ImplicitNamingStrategy & PhysicalNamingStrategy contracts in Hibernate. Implementations of these classes are used when mapping an Entity to database tables. These classes tell hibernate which table to use & which columns to use.