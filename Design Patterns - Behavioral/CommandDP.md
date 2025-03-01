## Command Design Pattern
**Why?**
### Represent a request or an operation on an object as an object
- We want to represent a request or a method call as an object. Information about parameters passed and the actual operation is encapsulated in a object called command.
- Advantage of command pattern is that, what would have been a method call is now an object which can be stored for later execution or sent to other parts of code.
- We can now even queue our command objects and execute them later.

### What is an Command?
- An Object that knows the command and receives an object to do the command on.

![UML](/Files/CommandDP.png)

### Implement a Command
- We start by writing command interface
- It must define method which executes the command
- We next implement this interface in class for each request or operation type we want to implement. Command should also allow for undo operation if your system needs it.
- Each concrete command knows exactly which operation it needs. All it needs is parameters for the operation if required and the receiver instance on which operation is invoked.
- Client creates the command instance and sets up receiver and all required parameters.
- The command instance is then ready to be sent to other parts of code. Invoker is the code that actually uses command instance and invokes the execute on the command.

### Implementation Considerations
- You can support "undo" & "redo" in your commands. This makes them really useful for systems with complex user interactions like workflow designers.
- If your command is simple i.e. if it doesn't have undo feature, doesn't have any state & simply hides a particular function & its arguments then you can reuse same command object for same type of request.
- For commands that are going to be queued for long durations, pay attention to size of state maintained by them.

### Design Considerations
- Commands can be inherited from other commands to reuse portions of code and build upon the base.
- You can also compose commands with other commands as well. These "macro" commands will have one or more sub-commands executed in sequence to complete a request.
- For implementing undo feature in your command you can make use of memento design pattern, which allows command to store the state information of receiver without knowing about internal objects used by receiver.

### xxx vs yyy DP

Command

- Command contains which operation is to be executed "by the receiver".
- Command encapsulates an action.

Strategy

- Strategy actually contains how the operation is to be carried out.
- Strategy encapsulates a particular algorithm.

### Pitfalls
- Things get a bit controversial when it comes to returning values & error handling with command.
- Error handling is difficult to implement without coupling the command with the client. In cases where client needs to know a return value of execution it's the same situation.
- In code where invoker is running in a different thread, which is very common in situations where command pattern is useful, error handling & return values get lot more complicated to handle.

### Examples:
- Distribution List of Email
- NOTE: receiver object can be optional as in SpringBoot we can use a @bean to get the receiver object.

![UML](/Files/CommandDPExample.png)

**Command Interface**
```java
package com.coffeepoweredcrew.command;

// Interface implemented by all concrete command classes
public interface Command {
	
	void execute();
}
```
**Add Member Command**
```java
package com.coffeepoweredcrew.command;

// A Concrete implementation of Command.
public class AddMemberCommand implements Command{
	
	private String emailAddress;
	
	private String listName;
	
	private EWSService receiver;
	
	public AddMemberCommand(String email, String listName, EWSService service) {
		this.emailAddress = email;
		this.listName = listName;
		this.receiver = service;
	}
	
	@Override
	public void execute() {
		receiver.addMember(emailAddress, listName);
	}
		
}
```
**Mail Task Runner**
```java
package com.coffeepoweredcrew.command;

import java.util.LinkedList;
import java.util.List;
// Throw Away POC code DON'T USE in PROD
// This is invoker actually executing commands. 
// Starts a worker thread in charge of executing commands
public class MailTasksRunner implements Runnable {

	private Thread runner;

	private List<Command> pendingCommands;

	private volatile boolean stop;

	private static final MailTasksRunner RUNNER = new MailTasksRunner();

	public static final MailTasksRunner getInstance() {
		return RUNNER;
	}

	private MailTasksRunner() {
		pendingCommands = new LinkedList<>();
		runner = new Thread(this);
		runner.start();
	}

	// Run method takes pending commands and executes them.
	@Override
	public void run() {

		while (true) {
			Command cmd = null;
			synchronized (pendingCommands) {
				if (pendingCommands.isEmpty()) {
					try {
						pendingCommands.wait();
					} catch (InterruptedException e) {
						System.out.println("Runner interrupted");
						if (stop) {
							System.out.println("Runner stopping");
							return;
						}
					}
				}
				cmd = pendingCommands.isEmpty()?null:pendingCommands.remove(0);
			}
			if (cmd == null)
				return;
			cmd.execute();
		}

	}
	
	// Giving it a command will schedule it for later execution
	public void addCommand(Command cmd) {
		synchronized (pendingCommands) {
			pendingCommands.add(cmd);
			pendingCommands.notifyAll();
		}
	}

	public void shutdown() {
		this.stop = true;
		this.runner.interrupt();
	}
}
```
**EWS Service**
```java
package com.coffeepoweredcrew.command;

// This class is the receiver.
public class EWSService {

	// Add a new member to mailing list
	public void addMember(String contact, String contactGroup) {
		// Contact exchange
		System.out.println("Added "+contact +" to "+contactGroup);
	}
	
	// Remove member from mailing list
	public void removeMember(String contact, String contactGroup) {
		// Contact exchange
		System.out.println("Removed "+contact +" from "+contactGroup);
	}
}
```
**Client Main**
```java
package com.coffeepoweredcrew.command;

public class Client {

	public static void main(String[] args) throws InterruptedException {
		EWSService service = new EWSService();
		
		Command c1 = new AddMemberCommand("a@a.com", "spam", service);
		MailTasksRunner.getInstance().addCommand(c1);
		
		Command c2 = new AddMemberCommand("b@b", "spam", service);
		MailTasksRunner.getInstance().addCommand(c2);
		
		Thread.sleep(3000); // Just so the example thread will have time to run
		MailTasksRunner.getInstance().shutdown();
	}

}
```

### Existing examples
- The java.lang.Runnable interface represents the Command pattern.
    - We create the object of class implementing runnable, providing all information it needs.
    - In the run method we'll call an operation on the receiver.
    - We can send this object for later execution to other parts of our application.
- The Action class in struts framework is also an example of Command pattern. Here each URL is mapped to a action class. We also configure the exact no-arg method in that class which is called to process that request.
