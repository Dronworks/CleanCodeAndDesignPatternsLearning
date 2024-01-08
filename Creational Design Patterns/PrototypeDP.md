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
- Note: when implementing the clone methode, consider shallow or deep copy

**Examples:**
- Local example:
  - A **GameUnit** prototype
  - A **Swordsman** that implements clone
  - A **General** that doesn't implement clone
  ![UML](/Files/PrototypeEmaple.png)