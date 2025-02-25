## Composite Design Pattern
**Why?**
### A part-whole relationship or hierarchy of objects, that are neede to be treated uniformly
- We have a part-whole relationship or hierarchy of objects and we want to be able to treat all objects in this hierarchy uniformly.
- This is NOT a simple composition concept from object oriented programming but a further enhancement to that principal.
- Think of composite pattern when dealing with tree structure of objects.

### What is an Composite?
- Working with child, composite is great for trees

![UML](/Files/CompositeDP.png)


### Implement a Composite
- We start by creating an abstract class/interface for Component
- Component must declare all methods that are applicable to both leaf and composite.
- We have to choose who defines the children management operations, component or composite.
- Then we implement the composite. An operation invoked on composite is propagated to all its children
- In leaf nodes we have to handle the non-applicable operations like add/remove a child if they are defined in component.
- In the end, a composite pattern implementation will allow you to write algorithms without worrying about whether node is leaf or composite.
- **Note:** This pattern doesn't show HOW to build a tree, but only how the tree will act.

### Implementation Considerations
- You can provide a method to access parent of a node. This will simplify traversal of the entire tree.
- You can define the collection field to maintain children in base component instead of composite but again that field has no use in leaf class.
- If leaf objects can be repeated in the hierarchy then shared leaf nodes can be used to save memory and initialization costs. But again the number of nodes is major deciding factor as using a cache for small total number of nodes may cost more.

### Design Considerations
- Decision needs to be made about where child management operations are defined. Defining them on component provides transparency but leaf nodes are forced to implement those methods. Defining them on composite is safer but client needs to be made aware of composite.
- Overall goal of design should be to make client code easier to implement when using composite. This is possible if client code can work with component interface only and doesn't need to worry about leaf-composite distinction.
- No need to cast to Leaf/Composite.

### Composite vs Decorator DP

Composite

- Deals with tree structure of objects.
- Leaf nodes and composites have same interface and composites simply delegate the operation to children.

Decorator
- Simply contains another (single) object.
- Decorators add or modify the behaviour of contained object and do not have notion of children.

### Pitfalls
- Difficult to restrict what is added to hierarchy. If multiple types of leaf nodes are present in system then client code ends up doing runtime checks to ensure the operation is available on a node.
- Creating the original hierarchy can still be complex implementation especially if you are using caching to reuse nodes and number of nodes are quite high.

### Examples:
- File system

![UML](/Files/CompositeFileSystemExample.png)

**Abstract File class - "Base"**
```java
//The component base class for composite pattern
//defines operations applicable both leaf & composite
public abstract class File {

    private String name;

    public File(String name) {
        this.name = name;
    }

    public String getName () {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public abstract void ls();
    public abstract void addFile(File file);
    public abstract File[] getFiles();
    public abstract boolean removeFile(File file);
}
```
**Bynary File Implementation - "Leaf node"**
```java
// Leaf node in composite pattern
public class BinaryFile extends File {

    private long size;

    public BinaryFile(String name, long size) {
        super (name) ;
        this.size = size;
    }

    @Override
    public void ls() {
        System.out.println(getName() + "t" + size);
    }

    // --------------------------------------------------
    // NOTE Next methods are not recommended being implemented here (as they only throw exception), but it can be a design decision
    // --------------------------------------------------
    @Override
    public void addFile(File file) {
        throw new UnsupportedOperationException("Leaf node doesn't support add operation");
    }

    @Override
    public File[] getFiles() {
        throw new UnsupportedOperationException("Leaf node doesn't support get operation");
    }

    @Override
    public boolean removeFile(File file) {
        throw new UnsupportedOperationException("Leaf node doesn't support remove operation");
    }
}
```
**Folder Implementation - "Composite"**
```java
//Composite in the composite pattern
public class Directory extends File {

    private List<File> children = new ArrayList<>();

    public Directory(String name) {
        super (name);
    }

    @Override
    public void ls() {
        System.out.println(getName());
        children.forEach(File::ls);
    }

    @Override
    public void addFile(File file) {
        children.add(file);
    }

    @Override
    public File[] getFiles() {
        return children.toArray(new File[children.size()]);
    }

    @Override
    public boolean removeFile(File file) {
        return children.remove(file);
    }
}
```
**Client Main class** 
```java
public class Client {

    public static void main(String[] args) {
        File root1 = createTreeOne();
        root1.ls();
        System.out.println(" ************* ");
        File root2 = createTreeTwo();
        root2.ls();
    }

    // Client builds tree using leaf and composites
    private static File createTreeOne() {
        File filel = new BinaryFile("Filel", 1000);
        Directory dir1 = new Directory("dir1");
        dir1.addFile(file1);
        File file2 = new BinaryFile("file2", 2000);
        File file3 = new BinaryFile("file3", 150);
        Directory dir2 = new Directory("dir2");
        dir2.addFile(file2);
        dir2.addFile(file3);
        dir2.addFile(dir1);
        return dir2;
    }

    private static File createTreeTwo() {
        return new BinaryFile("Another file", 200);
    }
}
```

### Existing examples
- Composite is used in many UI frameworks, since it easily allows to represent a tree of UI controls.
- In JSF we have UIViewRoot class which acts as composite. Other UIComponent implementations like UIOutput, UIMessage act as leaf nodes.

![IOExample](/Files/CompositeDPExternalExample.png)
