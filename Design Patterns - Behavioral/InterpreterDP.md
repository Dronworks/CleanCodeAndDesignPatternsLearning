## Interpreter Design Pattern
**Why?**
### Interprete a simple sentence (language) into a set of instructions
- We use interpreter when want to process a simple "language" with rules or grammar.
- E.g. File access requires user role and admin role.
- Interpreter allows us to represent the rules of language or grammar in a data structure and then interpret sentences in that language.
- Each class in this pattern represents a rule in the language. Classes also provide a method to interpret an expression.

### What is an Interpreter?
- Create an interpreter for a simple language.
- NOTE: The design pattern doesn't tell us HOW to build the TREE, but the RULES for interpreting a language.

![UML](/Files/InterpreterDP.png)


### Implement an Interpreter
- We start by studying rules of the language for which we want to build interpreter
    - We define an abstract class or interface to represent an expression & define a method in it which interprets the expression.
    - Each rule in the class becomes an expression. Expressions which do not need other expressions to interpret become terminal expressions.
    - We then create non-terminal expression classes which contain other expressions. These will in turn call interpret on children as well as perform interpretation of their own if needed.
- Building the abstract syntax tree using these classes can be done by client itself or we can create a separate class to do it.
- Client will then use this tree to interpret a sentence.
- A context is passed to interpreter. It typically will have the sentence to be interpreted & optionally it may also be used by interpreter to store any values which expressions need or modify or populate.

### Implementation Considerations
- Apart from interpreting expressions you can also do other things like pretty printing that use already built interpreter in new way.
- You still have to do the parsing. This pattern doesn't talk about how to actually parse the language & build the abstract syntax tree.
- Context object can be used to store & access state of the interpreter.

### Design Considerations
- You can use visitor pattern to interpret instead of adding interpret method in expression classes. Benefit of this is that if you are using multiple operations on the abstract syntax tree then visitor allows you to put these operations in a separate class.
- You can also use flyweight pattern for terminal expressions. You'll often find that terminal expressions can be reused.

### Interpreter vs Visitor DP

Interpreter

- Represents language rules or grammar as object structure.
- Has access to properties it needs for doing interpretation.

Visitor

- Represents operations to be performed on an object structure.
- We need an observable and observer functionality to gain access to the properties.

### Pitfalls
- Class per rule can quickly result in large number of classes, even for moderately complex grammar.
- Not really suitable for languages with complex grammar rules.
- This design pattern is very specific to a particular kind of problem of interpreting language.

### Examples:
- Permissions on a report file written in a simple language. E.g "admin can read and write, user can only read"

![UML](/Files/InterpreterDPExample.png)

**Report Class**
```java
package com.coffeepoweredcrew.interpreter;

public class Report {

	private String name;
	// "NOT ADMIN", "FINANCE_USER AND ADMIN"
	private String permission;
	
	public Report(String name, String permissions) {
		this.name = name;
		this.permission = permissions;
	}

	public String getName() {
		return name;
	}

	public String getPermission() {
		return permission;
	}

}
```
**User Class**
```java
package com.coffeepoweredcrew.interpreter;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Stream;

public class User {

	private List<String> permissions;
	
	private String username;
	
	public User(String username, String... permissions) {
		this.username = username;
		this.permissions = new ArrayList<>();
		Stream.of(permissions).forEach(e->this.permissions.add(e.toLowerCase()));
	}

	public List<String> getPermissions() {
		return permissions;
	}

	public String getUsername() {
		return username;
	}
	
}
```
**Permission Expression Interface**
```java
package com.coffeepoweredcrew.interpreter;

// Abstract expression
public interface PermissionExpression {

	boolean interpret(User user); 

}
```
**Permission Class - Terminal Expression**
```java
package com.coffeepoweredcrew.interpreter;

// Terminal expression
public class Permission implements PermissionExpression{

	private String permission;
	
	public Permission(String permission) {
		this.permission = permission.toLowerCase();
	}
	
	@Override
	public boolean interpret(User user) {
		return user.getPermissions().contains(permission);
	}

	@Override
	public String toString() {
		return permission;
	}

}
```
**AndExpression Class - Non-terminal Expression**
```java
package com.coffeepoweredcrew.interpreter;

// Non-terminal expression 
public class AndExpression implements PermissionExpression {

	private PermissionExpression expression1;

	private PermissionExpression expression2;
	
	public AndExpression(PermissionExpression expression1, PermissionExpression expression2) {
		this.expression1 = expression1;
		this.expression2 = expression2;
	}
	
	@Override
	public boolean interpret(User user) {
		return expression1.interpret(user) && expression2.interpret(user);
	}

	@Override
	public String toString() {
		return expression1 +" AND "+ expression2;
	}

}
```
**NotExpression Class - Non-terminal Expression**
```java
package com.coffeepoweredcrew.interpreter;

public class NotExpression implements PermissionExpression {
	
	private PermissionExpression expression;
	
	public NotExpression(PermissionExpression expression) {
		this.expression = expression;
	}

	@Override
	public boolean interpret(User user) {
		return !expression.interpret(user);
	}
	
	@Override
	public String toString() {
		return " NOT "+expression;
	}
}
```
**Or Expression Class - Non-terminal Expression**
```java
package com.coffeepoweredcrew.interpreter;

// Non terminal expression
public class OrExpression implements PermissionExpression {
	
	private PermissionExpression expression1;
	private PermissionExpression expression2;
	
	public OrExpression(PermissionExpression one, PermissionExpression two) {
		this.expression1 = one;
		this.expression2 = two;
	}

	@Override
	public boolean interpret(User user) {
		return expression1.interpret(user) || expression2.interpret(user);
	}
	
	@Override
	public String toString() {
		return expression1 +" OR "+expression2;
	}
}
```
**Expression Builder Class**
```java
package com.coffeepoweredcrew.interpreter;

import java.util.Stack;
import java.util.StringTokenizer;

// Parses & builds abstract syntax tree
public class ExpressionBuilder {
	private Stack<PermissionExpression> permissions = new Stack<>();

	private Stack<String> operators = new Stack<>();

	public PermissionExpression build(Report report) {
		parse(report.getPermission());
		buildExpressions();
		if (permissions.size() > 1 || !operators.isEmpty()) {
			System.out.println("ERROR!");
		}
		return permissions.pop();
	}

	private void parse(String permission) {
		StringTokenizer tokenizer = new StringTokenizer(permission.toLowerCase()); // Default tokenizer is space
		while (tokenizer.hasMoreTokens()) {
			String token;
			switch ((token = tokenizer.nextToken())) {
			case "and":
				operators.push("and");
				break;
			case "not":
				operators.push("not");
				break;
			case "or":
				operators.push("or");
				break;
			default:
				permissions.push(new Permission(token));
				break;
			}
		}
	}

	private void buildExpressions() {
		while (!operators.isEmpty()) {
			String operator = operators.pop();
			PermissionExpression perm1;
			PermissionExpression perm2;
			PermissionExpression exp;
			switch (operator) {
			case "not":
				perm1 = permissions.pop();
				exp = new NotExpression(perm1);
				break;
			case "and":
				perm1 = permissions.pop();
				perm2 = permissions.pop();
				exp = new AndExpression(perm1, perm2);
				break;
			case "or":
				perm1 = permissions.pop();
				perm2 = permissions.pop();
				exp = new OrExpression(perm1, perm2);
				break;
			default:
				throw new IllegalArgumentException("Unknown operator:" + operator);
			}
			permissions.push(exp);
		}
	}
}
```
**Client Main Class**
```java
package com.coffeepoweredcrew.interpreter;

public class Client {

	public static void main(String[] args) {
		Report report1  = new Report("Cashflow repot", "FINANCE_ADMIN OR ADMIN");
		ExpressionBuilder builder = new ExpressionBuilder();
		
		PermissionExpression  exp = builder.build(report1);
		System.out.println(exp);
		
		User user1 = new User("Dave", "USER");
		
		System.out.println("USer access report:"+ exp.interpret(user1));
	}

}
```

### Existing examples
- Regex
    - The java.util.regex.Pattern class is an example of interpreter pattern in Java class library.
    - Pattern instance is created with an internal abstract syntax tree, representing the grammar rules, during the static method call compile(). After that we check a sentence against this grammar using Matcher.

    ![IOExample](/Files/RegexInterpreterDPExample.png)

- Classes supporting the Unified Expression Language (EL) in JSP 2.1 JSF 1.2. These classes are in javax.el package. We have javax.el.Expression as a base class for value or method based expressions. We have javax.el.ExpressionFactory implementations to create the expressions. javax.el.ELResolver and its child classes complete the interpreter implementation.

- Cucumber testing engine uses interpreter pattern to interpret the feature files. It uses Gherkin parser to parse the feature files and then interprets them to run the tests.
