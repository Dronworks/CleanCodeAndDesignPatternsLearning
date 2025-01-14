### Singleton Design Pattern
**Why?**
#### To ensure that only a single instance of a class exists
- Any state that is added to the singleton becomes global state of the application

**What is a Singleton?**
- A calss that has only one instance, accessble globally through a single point.
![UML](/Files/SingletonDP.png)

**Implement a Singleton**
- Basics
    - Controlling instance creation
        - Class constructor(s) must be private
        - Subclassing/inheritance not allowed
    - Keeping track of instance
        - Class itself is a good place to track the instance
    - Giving access to the singleton instance
        - A public static method is good choice
        - Can expose instance as final public static field but it won't work for all singleton implementations
- Two options for implementing a singleton 
    - **Early initialization** - Eager Singleton (also spring beans)
        - Create singleton as soon as class is loaded
    - **Lazy initialization** - Lazy Singleton
        - Singleton is created when it is first required

**Down side of this Pattern**
- xxx

**Examples:**
- Eager Singleton
```
public class EagerRegistry {

    private EagerRegistry() {}

    private static final EagerRegistry INSTANCE = new EagerRegistry();

    public static EagerRegistry getInstance() {
        return INSTANCE;
    }
}
```
- Lazy Singleton

- Usage
```
public class Client {

    public static void main(String[] args) {
        EagerRegistry registry = EagerRegistry.getInstance();
        EagerRegistry registry2 = EagerRegistry.getInstance();
        System.out.println(registry == registry2);
    }
}
```