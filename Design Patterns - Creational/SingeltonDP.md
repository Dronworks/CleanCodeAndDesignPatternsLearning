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

 **Design Considerations**
 - Singleton doesn't need any parameters. If you need to support for constructor arguments, use simple factory or factory method.
 - Singleton should not carry mutable global state!

 **Singleton vs Factory Method**

Singleton
- Primary purpose or intent of singleton pattern is to ensure that only one instance of a class is ever created
- Singleton instance is created without any need of arguments from client code.

Factory Method
- Factory method is primarily used to isolate client code from object creation & delegate object creation to subclasses.
- Factory method allows to parameterize the object creation.

**Pitfalls**
- Hard to unit test as it hard to mock it.
 - Most common way of implementing is *per* CLASS LOADER and not per JVM so in a web app we may have 2 or more of same singleton.
 - Singleton shoud not have mutuble(changalbe) global states as it can be accessed through many places in code and it will have inconsistent behaviour.  Also this way it just become global variable. **Singleton** is best used as CONFIG, LOG, etc...

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

**Notes:**

1. **volatile** - makes it impossible to use cache
2. **synchronized** - makes the code be singlethreaded
```
public class LazyRegistryWithDCL {

    private LazyRegistryWithDCL() {}

    private static volatile LazyRegistryWithDCL INSTANCE;

    public static LazyRegistryWithDCL getInstance() {
        if(INSTANCE == null) {
            synchronized (LazyRegistryWithDCL.class) {
                if(INSTANCE == null) {
                    INSTANCE = new LazyRegistryWithDCL();
                }
            }
        }
        return INSTANCE;
    }
}
```
- Lazy Singleton via Holder. To avoid syncronize and **if** inside **if**
```
public class LazyRegistryIODH {

    private LazyRegistryIODH() {}

    private static class RegistryHolder {
        static LazyRegistryIODH INSTANACE = new LazyRegistryIODH();
    }

    public static LazyRegistryIODH getInstance () {
        return RegistryHolder. INSTANACE;
    }
}
```
- Singleton as an Enum - The only usage is that it is good for serialization. Its an antipattern.
```
public Enum RegistryEnum {
    INSTANCE;
    public void getConfiguration() {}
}
```
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

**Existing examples**
- java.lang.Runtime returns all the time same Runtime()
