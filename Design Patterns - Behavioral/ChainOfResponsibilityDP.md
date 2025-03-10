## Chain of Responsibility Design Pattern
**Why?**
### We want to avoid coupling the sender and the handler of a request. 
- xxx

### What is an xxx?
- We need to avoid coupling the code which sends request to the code which handles that request.
- Typically the code which wants some request handled calls the exact method on an exact object to process it, thus the tight coupling. Chain of responsibility solves this problem by giving more than one object, chance to process the request.
- We create objects which are chained together by one object knowing reference of object which is next in chain. We give request to first object in chain, if it can't handle that it simply passes the request down the chain.

![UML](/Files/ChainOfResponseDP.png)


### Implement a Chain if Responsibility
- We start by defining handler interface/abstract class
- Handler must define a method to accept incoming request
- Handler can define method to access successor in chain. If it's an abstract class then we can even maintain successor
- Next we implement handler in one or more concrete handlers. Concrete handler should check if it can handle the request. If not then it should pass request to next handler.
- We have to create our chain of objects next. We can do it in client. Typically in real world this job will be done by some framework or initialization code written by you.
- Client needs to know only the first object in chain. It'll pass on request to this object.

### Implementation Considerations
- Prefer defining handler as interface as it allows you to implement chain of responsibility without worrying about single inheritance rule of Java.
- Handlers can allow the request to propagate even if they handle the request. Servlet filter chains allow request to flow to next filter even if they perform some action on request.
- Chain can be described using XML or JSON as well so that you can add & remove handlers from chain without modifying code.

### Design Considerations
- Sometimes you can think of using existing connections or chains in objects. For example if you are using composite pattern you already have a chain which can be used to implement this behavior.

### Chain of Responsibility vs Command DP

Chain of Responsibility

- If handler can't handle the request it will pass it on to next handler.
- There is no guarantee that the request will be handled by at least one handler.
- We don't track which handler handled the request and can't reverse the actions of handler

Command

- With command there is no passing it on of request. Command handles the request itself.
- With command pattern it is assured that command will be executed and request will be handled.
- Commands are trackable. We can store command instances in same order as they execute and they are reversible in nature.

### Pitfalls
- There is no guarantee provided in the pattern that a request will be handled. Request can traverse whole chain and fall off at the other end without ever being processed and we won't know it.
- It is easy to misconfigure the chain when we are connecting successors. There is nothing in the pattern that will let us know of any such problems. Some handlers may be left unconnected to chain.

### Examples:
- Approve of Employee leave

![UML](/Files/ChainOfResponsibilityExample.png)

**Leave Application**
```java
package com.coffeepoweredcrew.chainofresponsibility;

import java.time.LocalDate;
import java.time.Period;
// Represents a request in our chain of responsibility
public class LeaveApplication {
	
	public enum Type {Sick, PTO, LOP};
	
	public enum Status {Pending, Approved, Rejecetd };
	
	private Type type;
	
	private LocalDate from;
	
	private LocalDate to;
	
	private String processedBy;

	private Status status;
	
	public LeaveApplication(Type type, LocalDate from, LocalDate to) {
		this.type = type;
		this.from = from;
		this.to = to;
		this.status = Status.Pending; 
	}
	
	public Type getType() {
		return type;
	}

	public LocalDate getFrom() {
		return from;
	}

	public LocalDate getTo() {
		return to;
	}

	public int getNoOfDays() {
		return Period.between(from, to).getDays();
	}
	
	public String getProcessedBy() {
		return processedBy;
	}

	public Status getStatus() {
		return status;
	}

	public void approve(String approverName) {
		this.status = Status.Approved;
		this.processedBy = approverName;
	}

	public void reject(String approverName) {
		this.status = Status.Rejecetd;
		this.processedBy = approverName;
	}
	
	public static Builder getBuilder() {
		return new Builder();
	}
	
	@Override
	public String toString() {
		return type + " leave for "+getNoOfDays()+" day(s) "+status
				+ " by "+processedBy;
	}
	public static class Builder {
		private Type type;
		private LocalDate from;
		private LocalDate to;
		private LeaveApplication application;
		
		private Builder() {
			
		}
		
		public Builder withType(Type type) {
			this.type = type;
			return this;
		}
		
		public Builder from(LocalDate from) {
			this.from = from;
			return this;
		}
		
		public Builder to(LocalDate to) {
			this.to = to;
			return this;
		}
		
		public LeaveApplication build() {
			this.application = new LeaveApplication(type, from, to);
			return this.application;
		}
		
		public LeaveApplication getApplication() {
			return application;
		}
	}
}
```
**ABSTRACT Class(we dont want anyone to create this object) Employee**
```java
package com.coffeepoweredcrew.chainofresponsibility;

// Abstract handler
public abstract class Employee implements LeaveApprover{

	private String role;
	
	private LeaveApprover successor;
	
	public Employee(String role, LeaveApprover successor) {
		this.role = role;
		this.successor = successor;
	}
	
	@Override
	public void processLeaveApplication(LeaveApplication application) {
		if(!processRequest(application) && successor != null) {
			successor.processLeaveApplication(application);
		}
	}

	protected abstract boolean processRequest(LeaveApplication application);
	
	@Override
	public String getApproverRole() {
		return role;
	}
	
}
```
**Leave Approver**
```java
package com.coffeepoweredcrew.chainofresponsibility;

// This represents a handler in chain of responsibility
public interface LeaveApprover {

	void processLeaveApplication(LeaveApplication application);
	
	String getApproverRole();
}
```
**Project Lead**
```java
package com.coffeepoweredcrew.chainofresponsibility;

// A concrete handler
public class ProjectLead extends Employee{

	public ProjectLead(LeaveApprover successor) {
		super("Project Lead", successor);
	}
	
	@Override
	protected boolean processRequest(LeaveApplication application) {
		// Type is sick leave & duration is less than or equal to 2 days
		if(application.getType() == LeaveApplication.Type.Sick) {
			if(application.getNoOfDays() <= 2) {
				application.approve(getApproverRole());
				return true;
			}
		}
		return false;
	}

}
```
**Manager**
```java
package com.coffeepoweredcrew.chainofresponsibility;

// A concrete handler
public class Manager extends Employee {

	public Manager(LeaveApprover nextApprover) {
		super("Manager", nextApprover);
	}
	
	@Override
	protected boolean processRequest(LeaveApplication application) {
		switch (application.getType()) {
		case Sick:
			application.approve(getApproverRole());
			return true;
		case PTO:
			if(application.getNoOfDays() <= 5) {
				application.approve(getApproverRole());
				return true;
			}
		}
		return false;
	}
	
}
```
**Director**
```java
package com.coffeepoweredcrew.chainofresponsibility;

import com.coffeepoweredcrew.chainofresponsibility.LeaveApplication.Type;
// A concrete handler
public class Director extends Employee {

	public Director(LeaveApprover nextApprover) {
		super("Director", nextApprover);
	}
	
	@Override
	protected boolean processRequest(LeaveApplication application) {
		if(application.getType() == Type.PTO) {
			application.approve(getApproverRole());
			return true;
		}
		return false;
	}
	
}
```
**Client Main code**
```java
package com.coffeepoweredcrew.chainofresponsibility;

import java.time.LocalDate;

import com.coffeepoweredcrew.chainofresponsibility.LeaveApplication.Type;

public class Client {

	public static void main(String[] args) {
	   LeaveApplication application = LeaveApplication.getBuilder().withType(Type.Sick)
			   	                      .from(LocalDate.now()).to(LocalDate.of(2018, 2, 28))
			   	                      .build();
	   System.out.println(application);
	   System.out.println("**************************************************");
	   LeaveApprover approver = createChain();
	   approver.processLeaveApplication(application);
	   System.out.println(application);
	}

	private static LeaveApprover createChain() {
		Director director = new Director(null);
		Manager manager = new Manager(director);
		ProjectLead lead = new ProjectLead(manager);
		return lead;
	}
	
}
```

### Existing examples
- Probably the best example of chain of responsibility is servlet filters. Each filter gets a chance to handle incoming request and passes it down the chain once its work is done.
- All servlet filters implement the javax.servlet.Filter interface which defines following doFilter method public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
- Implementations will use FilterChain object to pass request to next handler in chain i.e. chain.doFilter(request, response);
- In servlet filters, it's common practice to allow other filters to handle request even if current filter takes some action on the request.