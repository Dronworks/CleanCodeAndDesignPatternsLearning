## Visitor Design Pattern
**Why?**
### Add operations on an object without changeing the definition of the object
- Object (visitor) iterates over all data structure with another objects, and every other object calls a method from the visitor.
- Every time we need a new operation on visitor we will create a subclass.
- Objects have method "accept" to accept the visitor visit.


### What is a Visitor?
- A way to do actions on an objects, from another object with multiple implementations.

![UML](/Files/VisitorDP.png)


### Implement a Visitor
- Creating visitor interface where every function has a name of the class that the visitor will visit AND PARAMETER of this class.
- Objects that will use the visitor, have to have an "accept" method (interface or direct implementation). This method will call a visitor method for this class.
- Implement visitor interface with specific functionalities.

### Implementation Considerations
- Visitor can work with different classes (who **don't** have a **parent(interface/super)**). BUT the code that passes the Visitor needs to familiar with those classes.
- Often visitors will need the internal state of visiting classes, so we may need getters, setters.

### Design Considerations
- One effect of this pattern is that related functionality is grouped in a single visitor class instead of spread
across multiple classes. So adding new functionality is as simple as adding a new visitor class.
- Visitors can also accumulate state. So along with behavior we can also have state per object in our
visitor. We don't have to add new state to objects for behavior defined in visitor.
- Visitor can be used to add new functionality to object structure implemented using composite or can be
used for doing interpretation in interpreter design pattern.

### Visitor vs Strategy

Visitor

- All visitor subclasses provide possibly different functionalities from each other.

Strategy

- In strategy design pattern each subclasses represents a separate algorithm to solve the **same** problem.

### Pitfalls
- Often visitors need access to object's state. So we end up exposing a lot of state through getter methods,
weakening the encapsulation.
- Supporting a new class in our visitors requires changes to all visitor implementations.
- If the classes themselves change then all visitors have to change as well since they have to work with
changed class.
- A little bit confusing to understand and implement.

### Examples:
- Print with different algorithm implementation

![UML](/Files/VisitorDPExample.png)

**Employee class**
```java
package com. coffeepoweredcrew.visitor;
import java.util.Collection;
public interface Employee {
    int getPerformanceRating();
    void setPerformanceRating(int rating);
    Collection<Employee> getDirectReports ();
    int getEmployeeId();
    void accept(Visitor visitor);
}
```
**AbstractEmployee implements Employee**
```java
public abstract class AbstractEmployee implements Employee {
    private int performanceRating;
    private String name;
    private static int employeeIdCounter = 101;
    private int employeeId;
    
    public AbstractEmployee (String name) {
        this. name = name;
        employeeId = employeeIdCounter++;
    }
    
    public String getName () {
        return name;
    }
    
    @Override
    public int getPerformanceRating() {
        return performanceRating;
    }

    @Override
    public void setPerformanceRating(int rating) {
        performanceRating = rating;
    }

    @Override
    public Collection‹Employee> getDirectReports () {
        return Collections.emptyList();
    }

    @Override
    public int getEmployeeId () {
        return employeeld;
    }
}
```
**Programmer Employee**
```java
public class Programmer extends AbstractEmployee {
    private String skill;
   
    public Programmer(String name, String skill) {
        super (name) ;
        this.skill = skill;
    }

    public String getskill() {
        return skill;
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);|
    }
}
```
**ProjectLead Employee**
```java
public class ProjectLead extends AbstractEmployee {
    
    private List<Employee> directReports = new ArrayList‹>();
    
    public ProjectLead (String name, Employee... employees) {
        super (name) ;
        Arrays.stream(employees).forEach(directReports::add);
    }

    @Override
    public Collection‹Employee› getDirectReports() {
        return directReports;
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);|
    }
}
```
**ProjectLead Employee**
```java
public class Manager extends AbstractEmployee {
    
    private List<Employee> directReports = new ArrayList‹>();
    
    public Manager(String name, Employee... employees) {
        super(name);
        Arrays.stream(employees).forEach(directReports::add);
    }

    @Override
    public Collection‹Employee› getDirectReports() {
        return directReports;
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);|
    }
}
```
**VicePresident Class**
```java
public class VicePresident extends AbstractEmployee {
    
    private List<Employee> directReports = new ArrayList‹>();

    public VicePresident(String name, Employee... employees) {
        super(name);
        Arrays.stream(employees).forEach(directReports::add);
    }

    @Override
    public Collection‹Employee› getDirectReports() {
        return directReports;
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);|
    }
}
```
**Client Organization Class**
```java
public class Client {

    public static void main (String[] args) {
        Employee emps = buildOrganization();
        Visitor visitor = new PrintVisitor();
        visitOrgStructure(emps, visitor);|
    }

    private static Employee buildOrganization() {
        Programmer p1 = new Programmer ("Rachel", "C++");
        Programmer p2 = new Programmer ("Andy", "Java");

        Programmer p3 = new Programmer ("Josh", "C#");
        Programmer p4 = new Programmer ("Bill", "C++");

        ProjectLead pl1 = new ProjectLead("Tina", p1, p2);
        ProjectLead pl2 = new ProjectLead ("Joey", p3, p4) ;

        Manager m1 = new Manager ("Chad", pl1);
        Manager m2 = new Manager ("Chad II", pl2);

        VicePresident vp = new VicePresident("Richard", m1, m2) ;
        
        return vp;
    }

    private static void visitOrgStructure (Employee emp, Visitor visitor) {
        emp.accept(visitor);
        emp.getDirectReports().forEach(e -> visitOrgStructure(e, visitor));
    }

}
```
**Visitor interface**
```java
public interface Visitor {

    void visit(Programmer programmer); // Can be visitProgrammer, as ve need to implement accept method per class and it call call any "visit" method.
    void visit(ProjectLead lead);
    void visit(Manager manager);
    void visit(VicePresident vp);

}
```
**PrintVisitor implementation**
```java
public class PrintVisitor implements Visitor{
    @Override
    public void visit (Programmer programmer) {
        String msg = programmer.getName() +" is a " + programmer.getSkill()+" programmer.";
        System.out.println(msg);
    }

    @Override
    public void visit (ProjectLead lead) {
        String msg = lead.getName () +" is a Project Lead with "+lead.getDirectReports().size() +" programmers reporting. ";
        System.out.println(msg);
    }
    
    @Override
    public void visit (Manager manager) {
        String msg = manager.getName() +" is a Manager with "+manager.getDirectReports().size() +" leads reporting. ";
        System.out.println(msg) ;
    }
    
    @Override
    public void visit (VicePresident vp) {
        String msg = vp.getName() +" is a Vice President with "+vp. getDirectReports).size() +" managers reporting.";
        System.out.println(msg) ;
    }
}
```
**ApprisalVisitor implementation**
```java
public class AppraisalVisitor implements Visitor{
    private Ratings ratings = new Ratings ();
    
    @SuppressWarnings("serial")
    public class Ratings extends HashMap‹Integer, PerformanceRating>{
        public int getFinalRating(int empId) {
            return get(empId).getFinalRating();
        }
    }

    @Override
    public void visit (Programmer programmer) {
        PerformanceRating finalRating = new PerformanceRating(programmer.getEmployeeId(), programmer.getPerformanceRating());
        finalRating.setFinalRating(programmer.getPerformanceRating()) ;
        ratings.put(programmer.getEmployeeId(), finalRating);
    }

    @Override
    public void visit (ProjectLead lead) {
        //25% team & 75% personal
        PerformanceRating finalRating = new PerformanceRating(lead.getEmployeeId(), lead.getPerformanceRating());
        int teamAverage = getTeamAverage(lead);
        int rating = Math.round(0.75f * lead.getPerformanceRating() + o.25f*teamAverage);
        finalRating.setFinalRating(rating);
        finalRating. setTeamAverageRating(teamAverage);
        ratings.put(lead.getEmployeeId(), finalRating);
    }

    private int getTeamAverage(Employee emp) {
        return (int)Math.round(emp.getDirectReports().stream().mapToDouble(e -> e.getPerformanceRating()).average(). getAsDouble());
    }

    public Ratings getFinalRatings) {
        return ratings;
    }

}
```
### Existing examples
- The dom4j library used for parsing XML has interface org.dom4j.Visitor & implementation org.dom4j.VisitorSupport
which are examples of visitor. By implementing this visitor we can process each node in an XML tree.
- Another example of visitor pattern is the java.nio.file.FileVisitor & its implementation SimpleFileVisitor.

    ![EXAMPLE](/Files/VisitorFileExample.png)