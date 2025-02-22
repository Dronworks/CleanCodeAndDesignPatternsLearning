### Simple Factory Design Pattern
**Why?**
#### Move object creation logic to a separate class
- Create an instance during runtime, when we don't know what it will it be
- Subclasses will decide which object to instantiate by overriding the factory method
- No client code will be changed

**What is a simple factory?**
- Create classes that will create a different type of an object with the same interface
![UML](/Files/FactoryMethod.png)

**Implement a factory**
- Create a class for the creator (it can be concrete if provided a default object or abstract)
- Implementations will override the method and return an object
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