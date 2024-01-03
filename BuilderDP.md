### Builder Design Pattern
**Why?**
#### Creating an immutable class
One of the resons to have builder pattern is for immutable classes. This is usually solved by a constructor, but then it has to have many parameters, and if it is a library it is very hard to know what are the parameters.
#### A class that depends on different classes
`public User(String name, Address address, List<Role> roles)` - in this case Address and Roles need to be created befor the User.
#### A class with large constructors 
A class that have multiple fields and a constructor will have many arguments

**What is a builder?**
- If we have a complex process of creating an object with multiple steps
- In builder the logic of construction is removed from client code and abstracted in separate classes
- [UML](/Files/Builder.png)

**Implement a builder**
- Identify the parts of the product
- Provide mehods to create these parts
- Provide method to assemble the product
- Provide a way/method to get the build product. Also this can be a reference to avoid rebuilding
- Have a director class (or USUALLY a part of a class) that can do all the steps to build the product (usually the class that use this object)
```
private static UserDTO directBuild(UserDTOBuilder builder, User user) {
    return builder.withFirstName(user.getFirstName).
    //
    .with...
    //
    .build();
}
```
- Builder **could** be inside the building class as `public static class SomeBuilder{}`, Exclusions: if the builded class is an inheritance
- In the building class, the fields setter methods will be private, but builder inner class will have access to them
- In the building class, there is no need to a specific constructor and it will use setters
- Also we need to have a static method to get the Builder
`public static SomeBuilder getBuilder() { return new SomeBuilder() }`
#### Examples:
- **Calendar** is a good example of a builder - you can use some of its functions (like setWeekDate) and the builder will add locale and etc...
- **StringBuilder** - is a bad eample, althogh it works like a builder it doesn't follow the builder pattern [Link](https://stackoverflow.com/questions/5238007/stringbuilder-and-builder-pattern)
#### Nice example for builder:
```
interface ArticleBuilder {
  void addTitle(String title);
  void addParagraph(String paragraph);
}

void createArticle(ArticeBuilder articleBuilder) {
  articleBuilder.addTitle("Is String Builder an application of ...");
  articleBuilder.addParagraph("Is the Builder Pattern restricted...");
  articleBuilder.addParagraph("The StringBuilder class ...");
}
```
We may implement **ArticleBuilder** in 3 different ways and provide an **HtmlDocument** or a **TextDocument** or a **MarkdownDocument**