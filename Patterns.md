### Builder Design Pattern
**Why?**
#### Creating an immutable class
One of the resons to have builder pattern is for immutable classes. This is usually solved by a constructor, but then it has to have many parameters, and if it is a library it is very hard to know what are the parameters.
#### A class that depends on different classes
`public User(String name, Address address, List<Role> roles)` - in this case Address and Roles need to be created befor the User.

**What is a builder?**
- If we have a complex process of creating an object with multiple steps
- In builder the logic of construction is removed from client code and abstracted in separate classes
- [UML](/Files/Builder.png)