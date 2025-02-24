### Bridge Design Pattern
**Why?**
#### Decouple inheritances and implementations from base and interface
- Our implementations & abstractions are generally coupled to each other in normal inheritance.
- Using bridge pattern we can decouple them so they can both change without affecting each other.
- We achieve this feat by creating two separate inheritance hierarchies; one for implementation and another for abstraction.
- We use composition to bridge these two hierarchies.

**What is an Bridge?**
- Abstraction to tell us what we can acchieve
- Implementor to do the backgroud work to help to acchieve the abstraction
    - E.g. 
        - Abstraction: pull money from bankomat
        - Implementor: check pincode, validate cash amount, check dayli limit

- Inheritance of Abstract class with additional methods
- Abstract doesn't have to be Abstract class, but a method that hides the implementation (abstracts out the complexity of the implementation from the client)
- Then we can add an interface to define more methods, but not extend of the abstract class. For example checking accounts before bank transfer.
- RED POINT is the bridge

![UML](/Files/BridgeDP.png)


**Implement a Bridge**
- We start by defining our abstraction as needed by client
- We determine common base operations and define them in abstraction.
- We can optionally also define a refined abstraction & provide more specialized operations.
- Then we define our implementor next. Implementor methods do NOT have to match with abstractor. However abstraction can carry out its work by using implementor methods
- Then we write one or more concrete implementor providing implemention
- Abstractions are created by composing them with an instance of concrete implementor which is used by methods in abstraction.

 **Implementation Considerations**
- In case we are ever going to have a single implementation then we can skip creating abstract implementor.
- Abstraction can decide on its own which concrete implementor to use in its constructor or we can delegate that decision to a third class. In last approach abstraction remains unaware of concrete implementors & provides greater de-coupling.

**Design Considerations**
- Bridge provides great extensibility by allowing us to change abstraction and implementor independently. You can build & package them separately to modularize overall system.
- By using abstract factory pattern to create abstraction objects with correct implementation you can de-couple concrete implementors from abstraction.

**Bridge vs Adapter DP**

Bridge

- Intent is to allow abstraction and implementation to vary independently.
- Bridge has to be designed up front then only we can have varying abstractions & implementations.

Adapter

- Adapter is meant to make unrelated classes work together.
- Adapter finds its usage typically where a legacy system is to be integrated with new code.

**Pitfalls**
- It is fairly complex to understand & implement bridge design pattern.
- You need to have a well thought out & fairly comprehensive design in front of you before you can decide on bridge pattern.
- Needs to be designed up front. Adding bridge to legacy code is difficult. Even for ongoing project adding bridge at later time in development may require fair amount of rework.

**Examples:**
- Fifo collection
    - We only want to offer and poll. To acchive this we will have to have some DataStructure, that will act like this and have the methods to do so.
    - We can add to our abstraction more methods like peek, without affecting the DataStructure and vice versa.

![UML](/Files/BridgeDPExample.png)

**FifoCollection ABSTRACTION interface**
```java
//This is the abstraction.
//It represents a First in First Out collection
public interface FifoCollection<T> {

    //Adds element to collection
    void offer(T element);

    //Removes & returns first element from collection
    T poll();
}
```
**Queue<T> IMPLEMENT of FIFO**
```java
//A refined abstraction.
public class Queue<T> implements FifoCollection<T>{

    private LinkedList<T> list;

    public Queue(LinkedList<T> list) {
        this.list = list;
    }

    @Override
    public void offer(T element) {
        list.addLast(element);
    }

    @Override
    public T poll() {
        return list.removeFirst();
    }
}
```
**LinkedList IMPLEMENTOR INTERFACE**
```java
//This is the implementor.
//Note that this is also an interface
//As implementor is defining its own hierarchy which not related
//at all to the abstraction hierarchy.
public interface LinkedList<T> {

    void addFirst(T element);

    T removeFirst();

    void addLast(T element);

    T removeLast();

    int getSize();

}
```
**LinkedList IMPLEMENTOR IMPLEMENT**
```java
//A concrete implementor.
//This implementation is a classic LinkedList using nodes
// ** NOT thread safe **
public class SinglyLinkedList<T> implements LinkedList<T>{

    private class Node {
        private Object data;
        private Node next;
        private Node(Object data, Node next) {
            this.data = data;
            this.next = next;
        }
    }

    private int size;
    private Node top;
    private Node last;

    @Override
    public void addFirst(T element) {
        if(top == null) {
            last = top = new Node(element, null) ;
        } else {
            top = new Node(element, top);
        }
        size++;
    }

    }

    @Override
    public T removeFirst() {
        if(top == null) {
            return null;
        }
        @SuppressWarnings("unchecked")
        T temp = (T) top.data;
        if(top.next != nul) {
            top = top.next;
        } else {
            top = null;
            last = null;
        }
        size --;
        return temp;
    }
    ...
```
**Client Main class**
```java
public class Client {

    public static void main(String[] args) {
        FifoCollection<Integer> collection = new Queue<>(new SinglyLinkedList<>());
        collection.offer(10);
        collection.offer(40);
        collection.offer(99);

        System.out.println(collection.poll());
        System.out.println(collection.poll());
        System.out.println(collection.poll());

        System.out.println(collection.poll());
    }
}
```
**Existing examples**
- JDBC API. More specifically the java.sql.DriverManager class with the java.sql.Driver interface form a bridge pattern.
    - To use 
    ```java
    try (Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/myDb", "user1", "pass")) {
        // use con here
    }
    ```
    - The implementor (in this example SQL)
    ```java
    Class.forName("com.mysql.cj.jdbc.Driver");
    ```

    ![UML](/Files/JDBCBridgeDPExample.png)
- Collections.newSetFromMap() method
    
    ![Example](/Files/SetFromMapBridgeDPExample.png)
