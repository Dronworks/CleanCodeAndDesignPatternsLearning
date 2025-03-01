## Mediator Design Pattern
**Why?**
### Communicating state changes between objects
- Mediator encapsulates how a set of objects interact with each other. Due to this encapsulation there is a loose coupling between the interacting objects.
- Typically an object explicitly knows about other object to which it wants to interact i.e. to call a method. In mediator pattern this interaction is within the mediator object & interacting objects only know about the mediator object.
- Benefit of this arrangement is that the interaction can now change without needing modifications to participating objects. Changing the mediator allows to add/remove participants in an interaction.

### What is a Mediator?
- Object notifing other objects about a change in its state. 

![UML](/Files/MediatorDP.png)


### Implement a Mediator
- We start by defining mediator
    - Mediators define a generic method which is called by other objects.
    - This method typically needs to know which object changed and optionally the exact property which has changed in that object.
    - We implement this method in which we notify rest of the objects about the state change.
- Mediator needs to know about all participants in the collaboration it is mediating. To solve this problem we can either have objects register with mediator or mediator itself can be the creator of these objects
- Depending upon your particular implementation you may need to handle the infinite loop of change-notify-change which can result if object's value change handler is called for every value change whether from an external source as well as mediator
- NOTE: Defining characteristic of mediator is: it streamlines the communication between multiple objects. So a class which simply calls methods on multiple objects can't be a mediator confirming 100% to GoF mediator definition.

### Implementation Considerations
- It's important that mediator can identify which object has sent change notification to avoid sending that object the changed value again.
- If an object method took a very long time to process the change it can affect overall performance of mediator severely. In fact this is a common problem in any notification system, so pay attention to synchronization in mediator methods.
- We often end up with a complex mediator since it becomes a central point which ends up handling all routing between objects. This can make it a very difficult to maintain the mediator as the complexity grows.

### Design Considerations
- We can extend a mediator and create variations to be used in different situations like platform dependent interactions.
- Abstract mediator is often not required if the participating objects only work with that one mediator.
- We can use observer design pattern to implement the notification mechanism through which objects notify the mediator.

### Mediator vs Observer DP

Mediator

- Intent is to encapsulate complex interaction between objects
- Mediator implementations are typically specific to objects being mediated.

Observer

- Intent is to define one-to-many relationship between objects
- Observer pattern implementations are generic. Once implemented it can be used with any classes.

### Pitfalls
- Mediator becomes a central control object. As complexity of interaction grows, mediator complexity can quickly get out of hand.
- Making a reusable mediator, one which can be used with multiple sets of different objects is quite difficult. They are typically very specific to the collaboration. Another competing pattern called Observer is much more reusable.

### Examples:
- Ui updates

![UML](/Files/MediatorDPExample.png)

**UI controls**
```java
package com.coffeepoweredcrew.mediator;

// Abstract colleague
public interface UIControl {
	
	void controlChanged(UIControl control);
	
	String getControlValue();
	
	String getControlName();
}
```
**Label**
```java
package com.coffeepoweredcrew.mediator;

public class Label extends javafx.scene.control.Label implements UIControl{

	private UIMediator mediator;
	
	public Label(UIMediator mediator) {
		this.mediator = mediator;
		this.setMinWidth(100);
		this.setText("Label");
		mediator.register(this);
	}
	
	@Override
	public void controlChanged(UIControl control) {
		setText(control.getControlValue());
	}

	@Override
	public String getControlValue() {
		return getText();
	}

	@Override
	public String getControlName() {
		return "Label";
	}
	
}
```
**Slider**
```java
package com.coffeepoweredcrew.mediator;

public class Slider extends javafx.scene.control.Slider implements UIControl{

	private UIMediator mediator;
	private boolean mediatedUpdate;
	
	public Slider(UIMediator mediator) {
		this.mediator = mediator;
		setMin(0);
		setMax(50);
		setBlockIncrement(5);
		mediator.register(this);
		this.valueProperty().addListener((v,o,n) ->{if(!mediatedUpdate) this.mediator.valueChanged(this);});
	}
	
	@Override
	public void controlChanged(UIControl control) {
		mediatedUpdate = true;
		setValue(Double.valueOf(control.getControlValue()));
		mediatedUpdate = false;
	}

	@Override
	public String getControlName() {
		return "Slider";
	}

	@Override
	public String getControlValue() {
		return Double.toString(getValue());
	}
	
}
```
**TextBox**
```java
package com.coffeepoweredcrew.mediator;

import javafx.scene.control.TextField;

public class TextBox extends TextField implements UIControl{
	
	private UIMediator mediator;
	
	private boolean mediatedUpdate;
	
	public TextBox(UIMediator mediator) {
		this.mediator = mediator;
		this.setText("Textbox");
		this.mediator.register(this);

		this.textProperty().addListener((v, o, n) -> {
			if(!mediatedUpdate)
				this.mediator.valueChanged(this);
		});
	}

	@Override
	public void controlChanged(UIControl control) {
		this.mediatedUpdate = true;
		this.setText(control.getControlValue());
		this.mediatedUpdate = false;
	}

	@Override
	public String getControlValue() {
		return getText();
	}

	@Override
	public String getControlName() {
		return "Textbox";
	}
	
}
```
**UIMediator**
```java
package com.coffeepoweredcrew.mediator;

import java.util.ArrayList;
import java.util.List;

//Mediator
public class UIMediator {

	List<UIControl> colleagues = new ArrayList<>();
	
	public void register(UIControl control) {
		colleagues.add(control);
	}
	
	public void valueChanged(UIControl control) {
		colleagues.stream().filter(c-> c != control).forEach(c->c.controlChanged(control));
	}
}
```
**Client Main Class**
```java
package com.coffeepoweredcrew.mediator;

import javafx.application.Application;
import javafx.geometry.Insets;
import javafx.geometry.Pos;
import javafx.scene.Scene;
import javafx.scene.layout.GridPane;
import javafx.stage.Stage;

public class Client extends Application {

	@Override
	public void start(Stage primaryStage) throws Exception {
		UIMediator mediator = new UIMediator();
		Slider slider = new Slider(mediator);
		TextBox box = new TextBox(mediator);
		Label label = new Label(mediator);
		
		GridPane grid = new GridPane();
		grid.setAlignment(Pos.CENTER);
		grid.setVgap(20);
		grid.setPadding(new Insets(25, 25, 25, 25));
		grid.add(label, 0, 0);
		grid.add(slider, 0, 1);
		grid.add(box, 0, 2);
		Scene scene = new Scene(grid, 500, 500);
		primaryStage.setTitle("Mediator Pattern");
		primaryStage.setScene(scene);
		primaryStage.show();
	}

	public static void main(String[] args) {
		launch(args);
	}
}
```

### Existing examples
- The javax.swing.ButtonGroup class is an example of mediator. It takes care of making sure that only button in a group is selected. Participating "Buttons" notify this mediator when they are selected.

    ![IOExample](/Files/MediatorButtonExample.png)

- Sometimes a front controller is given as an example of mediator pattern. E.g. The DispatcherServlet in Spring.
- Purpose of front controller is to act as a central point where requests from outside world can land and then they are forwarded to appropriate page controller, often by use of some form of URL to class mapping.
- Front controller pattern can be thought of as a specialized version of mediator pattern. Front controller satisfies mediator characteristics like acting as central hub for communication between objects. It is specialized since it also handles requests from outside system & performs lookup to find a specific controller which will handle the request. In mediator when one object changes all others are notified!

    ![IOExample](/Files/FrontControllerMediatorExample.png)