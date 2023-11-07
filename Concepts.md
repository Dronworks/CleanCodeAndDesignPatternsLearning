### SOLID
Solid is a way to write good code. It is sort of guidance points.
- **Single Responsible Principle** 
    - A class focused, single functionality
    - A class addresses a specific concern
    - There should **never be more than one reason for a class to chnage**:
    ```
    Example:
    Lets say there is a server and super cool class that is communicating with this server. It has:
    - Json formatter
    - Http connection manager
    - And some authentication method

    But what if the connection is no longer an http?
    And tomorrow it is only XML?
    And the authentication suddenly is different?

    All this will lead to CHANGES IN THIS CLASS.
    ```
- **Open - Close Principle**
    - Software entities (classes, modules, methods, etc...) Should be **OPEN for extension, but CLOSE for modification**.
        - The Base code is already **WRITTEN AND TESTED**!
        - Also the **Base code is opened for extension**, and hence the new code car override it.
    - Example:
    ```
    A base class that hase some common fields like id, address, phone.
    Also add somve abstract methods. (behavior that will change between instances).
    An extension class that adds its own logic and fields.
    ```
- **Liskov Substitution Principle**
    - Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.
    - Extended class should not change the logic of a base class. 
    - Easier to explain in example:
    ```
    For example we have a Rectungle. It has height & width, and setters.
    A Square is a Rectungle with height = width.
    So we may think that a Square can extend a Rectungle and use it fields and functions (like Area).
    But by doing so we may rewrite the setters! as when setting height in Square we MUST set width to be the same!
    The problem: 
    If we test the set width and height (20 and 30), we expect to see 20 and 30! But we will see only 30 as for Square we changed the behavior of the Rectangle!

    To avoid this problem we may create a Shape interface and put there a function for Area. This was we wont brake base shapes.
    ```
    - If It Looks Like A Duck, Quacks Like A Duck, But Needs Batteries - You Probably Have The Wrong Abstraction.
    - Even more examples:
        - **Bad example:**
            ```
            public class Bird{
                public void fly(){}
            }
            public class Duck extends Bird{}
    
            The duck can fly because it is a bird, but what about this:

            public class Ostrich extends Bird{}
            ```
        - **Good example:**
            ```
            public class Bird{}
            public class FlyingBirds extends Bird{
                public void fly(){}
            }
            public class Duck extends FlyingBirds{}
            public class Ostrich extends Bird{} 
            ```
- **Interface Segregation Principle**
    - Clients should not be force to depend upon interfaces that they don't use
    - Interface pollution:
        - Large Interfaces
        - Unrelated Methods
    - Examples:
    ```
    Classes with empty method implementations.
    Methods with UnsupportedOperationException.
    Method implementation with nulls or dummy values.
    ```

- **Dependency Inversion Principle**
    - High level module SHOULD NOT depend on low level module
        - High level module - provides business rule
        - Low level module - functuanality so low level that can be used anywhere (like write on disk)
        - Don't make tightly couples
    - It should depend on Abstraction
        - **Instead** of using **New instance of FileWriter** we can create an **interface** and let the high level to use this **interface**
    - Example:
    ```
    Instead of
    writeMessage(Message message) {
        FileWriter writer = new FileWriter();
        writer.writeMessage(message);
    }

    Use:
    writeMessage(Message message, Writer writer) {
        writer.writeMessage(message);
    }
    ```

### Deisgn Patterns
- **Creational** - deal with the process of creation objects of classes.
- **Structural** - how we create or compose objects (add another object to an object and have a composition). How to arrange classes and objects so we can derive functionality from them.
- **Behavioral** - how classes and objects interact and communicate with each other. So we can derrive the functionality.