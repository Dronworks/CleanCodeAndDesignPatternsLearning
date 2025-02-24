### Object Pool Design Pattern
**Why?**
#### Cost of creating
- The cost of creating a instance might be high and we need a large number of objects for short duration.
- We can precreate all objects or collect unused objects in cache to reuse later.
- **NOTE:** this is good for SHORT DURATION USAGE!

**What is a Object Pool?**
- Create a reusable object
![UML](/Files/ObjectPoolDP.png)

**Implement a xxx**
- Create a class for the Object Pool
    - Thread-safe caching of objects
    - Methods to acquire and release objects should be provided. Pool should **reset** object before giving them.
- The reusable object must provide methods to reset its state upon "release" by code.
- We must decide whether to create a new pooled objects when pool is empty OR wait until an object is released. Choice can be easy if the object is tied to a fixed number of external resources.

**Implementation Considerations**
- Resetting object state should NOT be costly operation otherwise you may end up losing your performance savings.
- Pre-caching objects; meaning creating objects in advance can be helpful as it won't slow down the code using these objects. However it **may add-up to start up time & memory consumption**.
- Object pool's synchronization should consider the reset time needed & **avoid** resetting in **synchronized** context if possible.

**Design Consideration**
- Object pool can be parameterized to cache & return multiple objects and the acquire method can provide selection criteria. (like a factory)
- Pooling objects is only beneficial if they involve costly initialization because of initialization of external resource like a connection or a thread. Don't pool objects JUST to save memory, unless you are running into out of memory errors.
- Do not pool long lived objects or only to save frequent call to new. Pooling may actually negatively impact performance in such cases.

**REMEMBER**
- In now days object pools DON'T SAVE MEMORY ALLOCATION AND GC COST.
- This pattern is GREAT when interacting with **external resources** like **threads** and **connections**.

**Compare & Contrast with Prototype**

Object Pool
- We have cached objects that frequently live throughout programs entire run.
- Code using objects from object pool has to return the objects explicitly to pool. Depending on implementation, failing to return to pool may lead to memory and/or resource leak

Prototype
- Prototype creates object when needed and no caching is done.
- Once an object is cloned no special treatment is needed by client code and object can be used like any regular object.

**Pitfalls**
- Successful implementation depends on correct use by the client code (it is not promissed that we will implement it). Releasing objects back to pool can be vital for correct working.
- The reusable object needs to take care of resetting its state in efficient way. Some objects may not be suitable for pooling due to this requirement.
- Difficult to use in refactoring legacy code as the client code & reusable object both need to be aware of object pool.
- You have to decide what happens when pool is empty and there is a demand for an object. You can either wait for an object **BUT** waiting can have severe negative impact on performance, including deadlocks!
- If you create new objects when code asks for an object and none are available then you have **to do additional work** to maintain or trim the pool size or else you'll end up with very large pool.

**Examples:**
- Image object pool - A file **IO** for an image is costly, so we will have objects ready after a file IO.
![UML](/Files/ObjectPoolDP-Example.png)

**Image interface**
```java
import javafx.geometry. Point2D;
//Represents our abstract reusable
public interface Image extends Poolable {
    void draw() ;
    Point2D getLocation();
    void setLocation(Point2D location);
}
```
**Bitmap implemnentation**
```java
import javafx.geometry. Point2D;
// Concrete reusable - file name won't be changed
public class Bitmap implements Image {
    private Point2D location;
    private String fileName;
    public Bitmap (String fileName) {
        this.fileName = fileName;
    }

    @Override
    public void draw() {
        System.out.println("Drawing "+name+" @ "+location);
    }

    @Override
    public Point2D getLocation() {
        return location;
    }

    @Override
    public void setLocation(Point2D location) {
        this.ocation = location;
    }

    @Override // This is from Poolable interface
    public void reset() {
        location = null;
        System.out.println("location reset");
    }
}
```
**Pool interface**
```java
public interface Poolable {
    void reset();
}
```
**Object pool class**
```java
import java.util.concurrent. BlockingQueue;
import java.util. concurrent. LinkedBlockingQueue;

public class ObjectPool<T extends Poolable> {

    private BlockingQueue<T> availablePool;

    public ObjectPool(Supplier<T> creator, int count) {
        availablePool = new LinkedBlockingQueue<>();
        for(int i=0; i < count; i++) {
            availablePool.offer(creator.get()); // Prefill the pool
        }
    }
    
    public T get() {
        try {
            return availablePool. take();
        } catch (InterruptedException ex) {
            System.err.println("take() was interrupted"); // If pool is empty the blockable queue will interrupt
        }
        return null;
    }

    public void release(T obj) {
        obj.reset(); // This is the important part
        try {
            availablePool.put(obj);
        } catch (InterruptedException e) {
            System.err.println("put() was interrupted");
        }
    }
}
```
**Usage (client)**
```java
import javafx.geometry.Point2D;

public class Client {
    public static final ObjectPool<Bitmap> bitmapPool = new ObjectPool<>(()   -> new Bitmap("Logo.bmp"), 5);

    public static void main(String[] args) {
        Bitmap b1 = bitmapPool.get();
        bl.setLocation(new Point2D(10, 10));
        Bitmap b2 = bitmapPool.get();
        b2.setLocation(new Point2D(-10, 0));

        b1.draw();
        b2.draw();

        bitmapPool.release(b1);
        bitmapPool.release(b2);
    }
}
```

**Existing examples**
- java.util.concurrent.ThreadPoolExecutor - used by **newCachedThreadPooin()** via **Executor**
```java
ExecutorService service = Executors.newCachedThreadPool() ;

service.submit(()->System.out.println(Thread.currentThread().getName()));
service.submit(()->System.out.println(Thread.currentThread().getName()));
service.submit(()->System.out.println(Thread.currentThread().getName()));

service.shutdown() ;|
```
- org.apache.commons.dbcp.BasicDataSource - database connection pool

**NOTE:** next example is in pure java but also has a simple usage in springboot.
```java
// Construct BasicDataSource
//typically is bound to JNDI or set as spring bean
BasicDataSource dataSource = new BasicDataSource();
dataSource.setDriverClassName("com.mysql.jdbc. Driver") ;
dataSource.setUrl("jdbc:mysql://localhost/brooklyn_99_topsecret");
dataSource.setUsername("scully") ;
dataSource.setPassword("hitchcock");

//Then @ runtime
Connection conn = dataSource.getConnection();
//Use connection, and then close it to return it to pool.
conn.close();

//At application shutdown, close the pool
dataSource.close();
```