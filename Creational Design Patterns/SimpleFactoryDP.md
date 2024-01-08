### Factory Method Design Pattern
**Why?**
#### Types instatiation according to a simple creteria
- An if - else block for example
- We will use it if we have more than one option when instatiating an object

**What is a simple factory?**
- Simply move instatiation logic to a separate class and most commonly to a static method of this class
- Considered not to be real design pattern
- Learned as a destinction from Factory Method design pattern
- [UML](/Files/SimpleFactory.png)

**Implement a factory**
- An interface of the class
- Implementations of the class
- A simple factory class with a static function that returns the object (it may recieve additional arguments to use in instantiation)
- A client that calls the factory with a simple parameter

#### Examples:
- **java.text.NumberFormat** has a getInstance method that is an example