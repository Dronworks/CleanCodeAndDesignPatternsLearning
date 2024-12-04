
### Abstract Factory Design Pattern
**Why?**
#### Move object creation logic to a separate class
- Abstract factory is used when we have two or more objects which work together forming a kit or set and there can be multiple sets or kits that can be created by client code.
- So we separate client code from concrete objects forming such a set and also from the code which creates these sets.

**What is a simple factory?**

For example a game with different ages and in medival age: 
* a naval unit - is a viking sheep
* a soldier unit - is a viking

Now you progress to industrial age:
* a naval unit - is a stem ship
* a soldier unit - is a rifleman

Your class will ask for a naval unit, and the Factory will decide what age is it and build the unit accordingly, and return the class to you.

![UML](/Files/AbstractFactoryDP.png)

**Implement a factory**
- Create AbstractProductA and 2 implementations ProductAOpt1 and ProductAOpt2
- Create AbstractProductB and 2 implementations ProductBOpt1 and ProductBOpt2
- Create Abstract
- Also there is a possibility to provide to the function a simple parameter to select what message to build

**Down side of this Pattern**
- We will have to add more classes in order to create new types of interface object, hence more unit tests

**Examples:**
- One example is the java.util.Collecton, a function called **iterator()**
- Local example:
    - Create a Message -> once in Text format and once in JSON format
    ![UML](/Files/FactoryMethodExmple.png)
    - This is the Product (Message in our case)
    ```
    public abstract class Message {

        public abstract String getContent();

        public void addDefaultHeaders() {
            //Adds some default headers
        }

        public void encrypt() {
            //# Has some code to encrypt the content
        }
    }
    ```
    - This is a Message of type JSON
    ```
    public class JSONMessage extends Message {

        @Override
        public String getContent() {
            return "{\"JSON]\": []}";
        }
    }
    ```
    - This is the Message of Text type
    ```
    public class TextMessage extends Message {
        @Override
        public String getContent() {
            return "Text";
        }
    }
    ```
    - This is the abstract creator, typically it will do more logic on the created message
    ```
    package com.coffeepoweredcrew.factorymethod;
    import com.coffeepoweredcrew.factorymethod.message.Message;
        
    / **
    * This is our abstract "creator".
    * The abstract method createMessage() has to be implemented by
    * its subclasses.
    */
    public abstract class MessageCreator {

        public Message getMessage() {
            Message msg = createMessage();

            msg.addDefaultHeaders();
            msg.encrypt();

            return msg;
        }

        //Factory method
        public abstract Message createMessage();
    }
    ```
    - This is the implementation of JSON **Creator**
    ```
    public class JSONMessageCreator extends MessageCreator {
        @Override
        public Message createMessage() {
            return new JSONMessage();
        }
    }
    ```
    - This is the implementation of Text **Creator**
    ```
    public class TextMessageCreator extends MessageCreator {
        @Override
        public Message createMessage() {
            return new TextMessage();
        }
    }
    ```
    - Example of usage with Main
    ```
    public class Client {
        public static void main(String[] args) {
            printMessage(new JSONMessageCreator());
        }

        public static void printMessage(MessageCreator creator) {
            Message msg = creator. getMessage();
            System.out.println(msg);
        }
    }
    ```