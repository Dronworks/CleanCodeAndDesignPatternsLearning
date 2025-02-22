### Prototype Design Pattern
**Why?**
#### Creating multiple times an object that is costly to create
If the object that we are creating can take time to create and we need to create such object more than once

**What is a Prototype?**
- If we need to recreate an Object that is hard to create
![UML](/Files/PrototypeDP.png)

**Implement a prototype**
- Create a class that is a Prototype:
  - It implements Clonable
  - It should override clone() and return a copy of itself (it is good to create it public)
  - Not every subclass should implement it so the mothod should declare CloneNotSupportedException
- **Note:** when implementing the clone method, consider shallow or deep copy
  - If internal fields are Immutable shallow copy is good  
- **NOTE2:** cloneable is a marker interface, that only indicates that a class supports cloning

**Implementation considerations**
- Deep vs Shallow copy
- Don't forget to reset the mutual state of an object. Implement it in method so subclasses could implement this as well
- clone() is protected, it needs to be changed to public in order to use

**Design Consideration**
- Prototype are usefull when large objects with a majority of state is unchanged between instances, and the state is easily identifiable
- A **prototype registry** - a class where various prototypes can be used to clone.
- Prototype good for Composite and Decorator patterns

**Examples:**
- Local example:
  - A **GameUnit** prototype - base class that implements Cloneable
  - A **Swordsman** that implements clone
  - A **General** that doesn't implement clone
  ![UML](/Files/PrototypeEmaple.png)
  ```
  public abstract class GameUnit implements Cloneable {
    private Point3D position;

    public GameUnit() {
      position = Point3D. ZERO;
    }

    @Override
    public GameUnit clone() throws CloneNotSupportedException {
      GameUnit unit = (GameUnit)super.clone();
      unit. initialize();
      return unit;
    }

    protected void initialize() {
      this.position = Point3D.ZERO;
      reset();
    }

    protected abstract void reset();

    public GameUnit(float x, float y, float z) {
      position = new Point3D(x, y, z);
    }

    public void move(Point3D direction, float distance) {
      Point3D finalMove = direction.normalize();
      finalMove = finalMove.multiply(distance);
      position = position.add(finalMove);
    }

    public Point3D getPosition() {
      return position;
    }
  }
  ```
  ```
  public class Swordsman extends GameUnit {

    private String state = "idle";

    public void attack() {
      this. state = "attacking";
    }

    @Override
    public String toString() {
      return "Swordsman "+state+" @ "+getPosition();
    }
    
    @Override
    protected void reset() {
      state = "idle";
    }
  }
  ```
  ```
  public class General extends GameUnit{

    private String state = "idle";

    public void boostMorale() {
      this.state = "MoralBoost";
    }

    @override
    public String toString() {
      return "General "+state+" @ "+getPosition();
    }

    @Override
    public GameUnit clone() throws CloneNotSupportedException {
      throw new CloneNotSupportedException("Ganerals are unique");
    }

    @Override
    protected void reset() {
      throw new UnsupportedOperationException("Reset not supported");
    }
  }
  ```
  ```
  public class Client {

    public static void main(String[] args) {
      Swordsman sl = new Swordsman();
      sl.move(new Point3D(-10,0,0), 20);
      s1.attack();
      Swordsman s2 = (swordsman)s1.clone(); 
    }
  }
  ```