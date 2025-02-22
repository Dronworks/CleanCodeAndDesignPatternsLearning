### Adapter (sometimes Wrapper) Design Pattern
**Why?**
#### To support client code that expects different object that implements different interface
- We are adapting the object to client needs.
- Also called wrapper as it wraps an existing object.

**What is an Adapter?**
- An uml of typical adapter - can have many pitfalls

![UML](/Files/AdapterDP.png)
- **Better** option for Adapter UML (Object adapter)

![UML](/Files/ObjectAdapterDP.png)

**Implement a Adapter**
We start by creating a class for Adapter
- Adapter **must implement the interface** expected by client.
- **Option 1**: Class adapter - by also extending from client existing class. Simply going to forward the method to another method inherited from adaptee.
- **Option 2:** Object adapter - accept adaptee as constructor argument in adapter i.e. make use of composition.
- An object adapter should take adaptee as an argument in constructor or as a less preferred solution, you can instantiate it in the constructor thus tightly coupling with a specific adaptee.

 **Implementation Considerations**
- How much work the adapter does depends upon the differences between target interface and object being adapted. If method arguments are same or similar adapter has very less work to do.
- Using class adapter "allows" you to override some of the adaptee's behaviour. But this has to be **avoided** as you end up with adapter that behaves differently than adaptee. Fixing defects is not easy anymore!
- Using object adapter allows you to potentially change the adaptee object to one of its subclasses.

**Design Considerations**
- In java a "class adapter" may not be possible if both target and adaptee are concrete classes. In such cases the object adapter is the only solution. Also since there is no private inheritance in Java, it's better to stick with object adapter.
- A class adapter is also called as a two way adapter, since it can stand in for both the target interface and for the adaptee. That is we can use object of adapter where either target interface is expected as well as where an adaptee object is expected.

**Adapter vs Decorator DP**

Adapter

- Simply adapts an object to another interface without changing behaviour.
- Not easy to use recursive composition, that is an adapter adapting another adapter since adapters change interface 

Decorator

- Enhances object behaviour without changing its interface. 
- Since decorators do not change the interface, we can do recursive composition or in other words decorate a decorator with ease. Since a decorator is indistinguishable from main object

**Pitfalls**

Class adapter
- Expose the Base class methods as well as his own, that can lead to a user using both "getName" and "getFullName", what is not a best practice.
- We can extend only a single class.

General
- It is tempting to do additional things and not just adapt, but this can result in **different behaviour!** Fixing a defect in adapter will may not translate to the adaptee.

Summary: As long as we keep true to the purpose of simple interface translation we are good!

**Examples:**
- Buisness card Designer - CLASS Adapter
![UML](/Files/ClassAdapter.png)

    **Employee**
    ```
    public class Employee {
        private String fullName;
        private String jobTitle;
        private String officeLocation;

        ... public Getters, Setters ...
    }
    ```
    **Business Card Designer**
    ```
    / **
    * Client code which requires Customer interface.
    */
    public class BusinessCardDesigner
        public String designCard(Customer customer) {
            String card = "";
            card += customer.getName ();
            card += "\n" + customer.getDesignation();
            card += "\n" + customer.getAddress();
            return card;
        }
    }
    ```
    **Customer Interface**
    ```
    public interface Customer {
        String getName();
        String getDesignation();
        String getAddress();
    }
    ```
    **Employee Class Adapter**
    ```
    public class EmployeeClassAdapter extends Employee implements Customer{

        @Override
        public String getName() {
            return this.getFullName();
        }
    
        @Override
        public String getDesignation() {
            return this. getJobTitle();
        }
    
        @Override
        public String getAddress() {
            return this. getOfficeLocation();
        }
    }
    ```
    **Main**
    ```
    public class Main {

        public static void main(String[] args) {
            / ** Using Class/Two-way adapter ** /
            EmployeeClassAdapter adapter = new EmployeeClassAdapter();
            populateEmployeeDato(adapter);
            BusinessCardDesigner designer = new BusinessCardDesigner();
            String card = designer.designCard(adapter);
            System.out.println(card);

            / ** Using Object Adapter ** /
            Employee employee = new Employee();
            populoteEmployeeData(employee);
            EmployeeObjectAdapter objectAdapter = new Employee0bjectAdapter (employee);
            card = designer.designCard(objectAdapter);
            System.out.println(card);
        }

        private static void populateEmployeeData (Employee employee) {
            employee.setFullName("Elliot Alderson");
            employee.setJobTitle("Security Engineer");
            employee.setOfficeLocation("Allsafe Cybersecurity, New York City, New York");
        }
    }
    ```


- Business card Designer - OBJECT Adapter
![UML](/Files/ObjectAdapter.png)
    
    **Employee Object Adapter**
    ```
    public class EmployeeObjectAdapter implements Customer{

        private Employee adaptee;

        public EmployeeObjectAdapter (Employee adaptee) {
            this.adaptee = adaptee;
        }

        @Override
        public String getName () {
            return adaptee. getFullName ();
        }

        @Override
        public String getDesignation() {
            return adaptee.getJobTitle();
        }

        @Override
        public String getAddress() {
            return adaptee.get0fficeLocation();
        }
    }
    ```



**Existing examples**
- The java.io.InputStreamReader and java.io.OutputStreamWriter classes are examples of object adapters.
- These classes adapt existing InputStream/OutputStream object to a Reader/Writer interface.
```
public class InputStreamReader extends Reader {
    private final StreamDecoder sd;

    public InputStreamReader(InputStream in) {
        super (in);
        try {
            sd = StreamDecoder.forInputStreamReoder(in, this, (Str
        } catch (UnsupportedEncodingException e) {
            // The default encoding should always be available
            throw new Error(e);
        }
    }

    public int read(char cbuf[], int offset, int length) throws IOE {
        return sd.read(cbuf, offset, length);
    }

    public int read() throws IOException { 
        return sd.read();
    }
}
```