## Iterator Design Pattern
**Why?**
### Accessing elements or children of an object without exposing its internal structure
- Iterator allows a way to access elements/children of an aggregate object in sequence while hiding the actual internal data structure used.
- In Java language iterators are integral part of collection frameworks and they are implementations of this design pattern.
- Iterators are stateful, meaning an iterator object remembers its position while iterating. 
- Iterators can become out of sync if the underlying collection is changed while a code is using iterator.

### What is an Iterator?
- Getting object after object without exposing the internal structure of the collection.

![UML](/Files/IteratorDP.png)


### Implement an Iterator
- We start by defining Iterator interface
- Iterator has methods to check whether there is an element available in sequence & to get that element
- We then implement the Iterator in a class. This is typically an inner class in our concrete aggregate. Making it an inner class makes it easy to access internal data structures
- Concrete iterator needs to maintain state to tell its position in collection of aggregate. If the inner collection changes it can throw an exception to indicate invalid state.

### Implementation Considerations
- Detecting change to underlying data structure while some code is using an iterator is important to notify to the client because then our iterator may not work correctly.
- Having our iterator implementation as inner class makes it easy to access internal collection of aggregate objects.

### Design Considerations
- Always prefer iterator interface so you can change the implementation without affecting client.
- Iterators have many applications where a collection is not directly used but we still want to give a sequential access to information for example may be for reading lines from file, from network.

### Pitfalls
- Access to index during iteration is not readily available like we have in a for loop.
- Making modifications to the collection while someone is using an iterator often makes that iterator instance invalid as its state may not be valid.

### Examples:
- Iterator on Enum

![UML](/Files/IteratorOnEmum.png)

**Iterator interface**
```java
package com.coffeepoweredcrew.iterator;

// Iterator interface allowing to iterate over values of an aggregate
public interface Iterator<T> {

	boolean hasNext();
	
	T next();
}
```
**Theme color iterator**
```java
package com.coffeepoweredcrew.iterator;

// This enum represents the aggregate from iterator pattern
public enum ThemeColor {

	RED,
	ORANGE,
	BLACK,
	GREEN,
	WHITE;

	public static Iterator<ThemeColor> getIterator() {
		return new ThemeColorIterator();
	}
	
	private static class ThemeColorIterator implements Iterator<ThemeColor> {
		
		private int position;

		@Override
		public boolean hasNext() {
			return position < ThemeColor.values().length;
		}

		@Override
		public ThemeColor next() {
			return ThemeColor.values()[position++];
		}
		
	}
}
```
**Client code**
```java
package com.coffeepoweredcrew.iterator;

public class Client {

	public static void main(String[] args) {
		Iterator<ThemeColor> iter = ThemeColor.getIterator();
		while(iter.hasNext()) {
			System.out.println(iter.next());
		}
	}

}
```

### Existing examples
- The iterator classes in Java's collection framework are great examples of iterator. C The concrete iterators are typically inner classes in each collection class implementing java.util.Iterator interface.
- The java.util.Scanner class is also an example of Iterator pattern. This class supports InputStream as well and allows to iterate over a stream.
    ```java
    Scanner sc = new Scanner(System.in);
    //read an integer from input stream
    int i = sc.nextInt();
    ```
- The javax.xml.stream.XMLEventReader class is also an iterator. This class turns the XML into a stream of event objects.
