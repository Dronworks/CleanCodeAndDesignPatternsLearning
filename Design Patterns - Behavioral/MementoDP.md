## Memento Design Pattern
**Why?**
### Store object state without exposing its implementation details. Snapshot of an object's state are taken constantly and can be restored later.
- When we want to store object's state without exposing internal details about the state then we can use memento design pattern.
- The main intent behind saving state is often because we want to restore the object to a saved state.
- Using memento we can ask an object to give its state as a single, "sealed" object & store it for later use. This object should not expose the state for modification.
- This pattern is often combined with Command design pattern to provide undo functionality in application.

### What is a Memento?
- Memento is a behavioral design pattern that lets you save and restore the previous state of an object without revealing the details of its implementation.

![UML](/Files/MementoDP.png)


### Implement a Memento
- We start by finding originator state which is to be "stored" in memento.
- We then implement the memento with requirement that it can't be changed & read outside the originator.
- Originator provides a method to get its current snapshot out, which will return an instance of memento.
- Another method in originator takes a memento object as argument and the originator object resets itself to match with the state stored in memento.

### Implementation Considerations
- It is important to keep an eye on the size of state stored in memento. A solution for discarding older state may be needed to handle large memory consumption scenarios.
- Memento often ends up being an inner class due to the requirement that it must encapsulate ALL details of what is stored in its instance.
- Resetting to previous state should consider effects on states of other objects/services.

### Design Considerations
- If there is a definite, fixed way in which mementos are created then we can only store incremental state in mementos. This is especially true if we are using command design pattern where every command stores a memento before execution.
- Mementos can be stored internally by originator as well but this complicates the originator. An external caretaker with fully encapsulated Memento provides you with more flexibility in implementation.

### Memento vs Command DP

Memento

- State of memento is sealed for everyone except originator
- A memento needs to be stored for it to be of any use.

Command

- Although commands are typically immutable their state is often readable.
- Commands can be stored as well but not storing them after execution is optional.


### Pitfalls
- In practice creating a snapshot of state may not be easy if other objects are part of originator's state.
- Resetting a state may not be as simple as copying references. If state change of originator is tied with other parts of application then those parts may become out of sync/invalid due to resetting state.

### Examples:
- Workflow creator

![UML](/Files/MementoDPExample.png)

**Workflow**
```java
package com.coffeepoweredcrew.memento;

import java.util.Arrays;
import java.util.LinkedList;
import java.util.stream.Collectors;

public class Workflow {

	private LinkedList<String> steps;
	
	private String name;
	
	public Workflow(String name) {
		this.name = name;
		this.steps = new LinkedList<>();
	}
	
	public Workflow(String name, String... steps) {
		this.name = name;
		this.steps = new LinkedList<>();
		if(steps != null && steps.length > 0) {
			Arrays.stream(steps).forEach(s->this.steps.add(s));
		}
	}

	@Override
	public String toString() {
		StringBuilder builder = new StringBuilder("Workflow [name=");
		builder.append(name).append("]\nBEGIN -> ");
		for(String step : steps) {
			builder.append(step).append(" -> ");
		}
		builder.append("END");
		return builder.toString();
	}
	
	public void addStep(String step) {
		steps.addLast(step);
	}
	
	public boolean removeStep(String step) {
		return steps.remove(step);
	}

	public String[] getSteps() {
		return steps.toArray(new String[steps.size()]);
	}

	public String getName() {
	    return name;
    }
}
```
**Workflow Designer (with memenot class inside)**
```java
package com.coffeepoweredcrew.memento;

public class WorkflowDesigner {

    private Workflow workflow;

    public void createWorkflow(String name) {
        workflow = new Workflow(name);
    }

    public Workflow getWorkflow() {
        return this.workflow;
    }

    public Memento getMemento() {
       if(workflow == null) {
    	   return new Memento();
       }
       return new Memento(workflow.getSteps(), workflow.getName());
    }

    public void setMemento(Memento memento) {
    	if(memento.isEmpty()) {
    		this.workflow = null;
    	} else {
    		this.workflow = new Workflow(memento.getName(), memento.getSteps());
    	}
    }

    public void addStep(String step) {
        workflow.addStep(step);
    }

    public void removeStep(String step) {
        workflow.removeStep(step);
    }

    public void print() {
        System.out.println(workflow);
    }
    // Memento
    public class Memento {
    	
    	private String[] steps;
    	
    	private String name;
    	
    	private Memento() {
    		
    	}
    	
    	private Memento(String[] steps, String name) {
    		this.steps = steps;
    		this.name = name;
    	}
    	
    	private String[] getSteps() {
    		return steps;
    	}
    	
    	private String getName() {
    		return name;
    	}
    	
    	private boolean isEmpty() {
    		return this.getSteps() == null && this.getName() == null;
    	}
    }
        
}
```
**AbstractWorkflowCommand**
```java
package com.coffeepoweredcrew.memento.command;

import com.coffeepoweredcrew.memento.WorkflowDesigner;

public abstract class AbstractWorkflowCommand implements WorkflowCommand {

    protected WorkflowDesigner.Memento memento;

    protected WorkflowDesigner receiver;

    public AbstractWorkflowCommand(WorkflowDesigner designer) {
        this.receiver = designer;
    }

    @Override
    public void undo() {
        receiver.setMemento(memento);
    }
}
```
**AddStepCommand**
```java
package com.coffeepoweredcrew.memento.command;

import com.coffeepoweredcrew.memento.WorkflowDesigner;

public class AddStepCommand extends AbstractWorkflowCommand {

    private String step;

    public AddStepCommand(WorkflowDesigner designer, String step) {
        super(designer);
        this.step = step;
    }

    @Override
    public void execute() {
        this.memento = receiver.getMemento();

        receiver.addStep(step);
    }
}
```
**Create Command**
```java
package com.coffeepoweredcrew.memento.command;

import com.coffeepoweredcrew.memento.WorkflowDesigner;

public class CreateCommand extends AbstractWorkflowCommand {

    private String name;

    public CreateCommand(WorkflowDesigner designer, String name) {
        super(designer);
        this.name = name;
    }

    @Override
    public void execute() {
        this.memento = receiver.getMemento();
        receiver.createWorkflow(name);
    }

}
```
**RemoveStepCommand**
```java
package com.coffeepoweredcrew.memento.command;

import com.coffeepoweredcrew.memento.WorkflowDesigner;

public class RemoveStepCommand extends AbstractWorkflowCommand {

    private String step;

    public RemoveStepCommand(WorkflowDesigner designer, String step) {
        super(designer);
        this.step = step;
    }

    @Override
    public void execute() {
        memento = receiver.getMemento();
        receiver.removeStep(step);
    }
}
```
**WorkflowCommand**
```java
package com.coffeepoweredcrew.memento.command;

public interface WorkflowCommand {

    void execute();

    void undo();
}
```
**Client**
```java
package com.coffeepoweredcrew.memento;

import com.coffeepoweredcrew.memento.command.AddStepCommand;
import com.coffeepoweredcrew.memento.command.CreateCommand;
import com.coffeepoweredcrew.memento.command.WorkflowCommand;

import java.util.Collection;
import java.util.LinkedList;

public class Client {

    public static void main(String[] args) {
        WorkflowDesigner designer = new WorkflowDesigner();
        LinkedList<WorkflowCommand> commands = runCommands(designer);
        designer.print();
        commands.removeLast().undo();
        designer.print();
        commands.removeLast().undo();
        designer.print();

    }

    private static void undoLastCommand(LinkedList<WorkflowCommand> commands) {
        if(!commands.isEmpty())
            commands.removeLast().undo();
    }

    private static LinkedList<WorkflowCommand> runCommands(WorkflowDesigner designer) {
        LinkedList<WorkflowCommand> commands = new LinkedList<>();

        WorkflowCommand cmd = new CreateCommand(designer,"Leave Workflow");
        commands.addLast(cmd);
        cmd.execute();

        cmd = new AddStepCommand(designer,"Create Leave Application");
        commands.addLast(cmd);
        cmd.execute();

        cmd = new AddStepCommand(designer,"Submit Application");
        commands.addLast(cmd);
        cmd.execute();

        cmd = new AddStepCommand(designer,"Application Approved");
        commands.addLast(cmd);
        cmd.execute();

        return commands;
    }
}
```



### Existing examples
- One great example of memento is the undo support provided by the javax.swing.text.JTextComponent and its child classes like JTextField, JTextArea etc.
- The javax.swing.undo.UndoManager acts as the caretaker & implementations of javax.swing.undo.UndoableEdit interface work as mementos. The javax.swing.text.Document implementation which is model for text components in swing is the originator.

    ```java
    // Create & attach UndoManager to a text field
    UndoManager undoManager = new UndoManager();
    JTextField text = new JTextField(10);
    text.getDocument().addUndoableEditListener(undoManager);

    //  After some edits
    if(undoManager.canUndo()) {
        undoManager.undo();
    }
    ```
 **NOT AN EXAMPLE**
 
 The java.io.Serializable is often given as an example of Memento but it is NOT a memento & not even a design pattern. Memento object itself can be serialized but it is NOT mandatory requirement of the pattern. In fact mementos work most of the time times with in-memory snapshots of state.
