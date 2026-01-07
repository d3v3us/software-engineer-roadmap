# Software Design Patterns - Behavioral Deep Dive - Complete Understanding

## Table of Contents
1. [What are Behavioral Patterns?](#what-are-behavioral-patterns)
2. [Why Behavioral Patterns Matter](#why-behavioral-patterns-matter)
3. [Observer Pattern](#observer-pattern)
4. [Strategy Pattern](#strategy-pattern)
5. [Command Pattern](#command-pattern)
6. [Chain of Responsibility](#chain-of-responsibility)
7. [State Pattern](#state-pattern)
8. [Template Method Pattern](#template-method-pattern)
9. [Best Practices](#best-practices)

---

## What are Behavioral Patterns?

### Definition

**Behavioral Patterns**: Design patterns for communication between objects.

**Key Concepts:**
- **Communication**: Object communication
- **Responsibilities**: Object responsibilities
- **Algorithms**: Algorithm organization
- **Flexibility**: Behavioral flexibility

### Real-World Analogy

**Behavioral Patterns = Communication Protocols:**
- **People**: Objects
- **Communication**: Object communication
- **Protocols**: Behavioral patterns
- **Coordination**: System coordination

**Software:**
- **Objects**: Software objects
- **Communication**: Object communication
- **Patterns**: Behavioral patterns
- **Coordination**: System coordination

---

## Why Behavioral Patterns Matter?

### Impact of Communication

**1. Flexibility:**
```
Flexible communication
  ↓
Easy changes
  ↓
Adaptability
```

**2. Decoupling:**
```
Loose coupling
  ↓
Independent objects
  ↓
Maintainability
```

**3. Reusability:**
```
Reusable behaviors
  ↓
Code reuse
  ↓
Efficiency
```

### Benefits of Behavioral Patterns

**1. Flexibility:**
- **Behavioral flexibility**: Flexible object behavior
- **Runtime changes**: Runtime behavior changes
- **Extensibility**: Easy to extend

**2. Decoupling:**
- **Loose coupling**: Loose object coupling
- **Independence**: Object independence
- **Maintainability**: Easier maintenance

**3. Reusability:**
- **Behavior reuse**: Reuse behaviors
- **Code reuse**: Code reusability
- **Efficiency**: More efficient

---

## Observer Pattern

### What is Observer Pattern?

**Observer Pattern**: Define one-to-many dependency between objects.

**Use when:**
- **State changes**: Object state changes
- **Notifications**: Need to notify multiple objects
- **Event handling**: Event-driven systems

### Observer Implementation

**Observer Example:**
```java
// Subject
public interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

public class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private int state;
    
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(state);
        }
    }
    
    public void setState(int state) {
        this.state = state;
        notifyObservers();
    }
}

// Observer
public interface Observer {
    void update(int state);
}

public class ConcreteObserver implements Observer {
    private int state;
    
    public void update(int state) {
        this.state = state;
        System.out.println("Observer updated: " + state);
    }
}
```

### Observer Benefits

**1. Decoupling:**
```
Subject and observers
  ↓
Loose coupling
  ↓
Flexibility
```

**2. Notifications:**
```
State changes
  ↓
Automatic notifications
  ↓
Event handling
```

---

## Strategy Pattern

### What is Strategy Pattern?

**Strategy Pattern**: Define family of algorithms and make them interchangeable.

**Use when:**
- **Multiple algorithms**: Multiple ways to do something
- **Runtime selection**: Select algorithm at runtime
- **Avoid conditionals**: Avoid conditional statements

### Strategy Implementation

**Strategy Example:**
```java
// Strategy
public interface PaymentStrategy {
    void pay(int amount);
}

public class CreditCardStrategy implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardStrategy(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    public void pay(int amount) {
        System.out.println(amount + " paid with credit card");
    }
}

public class PayPalStrategy implements PaymentStrategy {
    private String email;
    
    public PayPalStrategy(String email) {
        this.email = email;
    }
    
    public void pay(int amount) {
        System.out.println(amount + " paid using PayPal");
    }
}

// Context
public class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}
```

### Strategy Benefits

**1. Flexibility:**
```
Multiple algorithms
  ↓
Runtime selection
  ↓
Flexibility
```

**2. Extensibility:**
```
New strategies
  ↓
Easy to add
  ↓
Open/Closed Principle
```

---

## Command Pattern

### What is Command Pattern?

**Command Pattern**: Encapsulate request as object.

**Use when:**
- **Request queuing**: Queue requests
- **Undo/Redo**: Implement undo/redo
- **Logging**: Log requests

### Command Implementation

**Command Example:**
```java
// Command
public interface Command {
    void execute();
    void undo();
}

public class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    public void execute() {
        light.turnOn();
    }
    
    public void undo() {
        light.turnOff();
    }
}

// Invoker
public class RemoteControl {
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
    }
}
```

### Command Benefits

**1. Decoupling:**
```
Request and receiver
  ↓
Decoupled
  ↓
Flexibility
```

**2. Undo/Redo:**
```
Command history
  ↓
Undo/redo support
  ↓
User experience
```

---

## Chain of Responsibility

### What is Chain of Responsibility?

**Chain of Responsibility**: Pass request along chain of handlers.

**Use when:**
- **Multiple handlers**: Multiple objects can handle request
- **Handler selection**: Select handler at runtime
- **Request processing**: Process requests in chain

### Chain of Responsibility Implementation

**Chain Example:**
```java
// Handler
public abstract class Handler {
    protected Handler next;
    
    public void setNext(Handler next) {
        this.next = next;
    }
    
    public abstract void handleRequest(Request request);
}

public class ConcreteHandler1 extends Handler {
    public void handleRequest(Request request) {
        if (request.getType() == RequestType.TYPE1) {
            // Handle request
        } else if (next != null) {
            next.handleRequest(request);
        }
    }
}

public class ConcreteHandler2 extends Handler {
    public void handleRequest(Request request) {
        if (request.getType() == RequestType.TYPE2) {
            // Handle request
        } else if (next != null) {
            next.handleRequest(request);
        }
    }
}
```

---

## State Pattern

### What is State Pattern?

**State Pattern**: Allow object to alter behavior when internal state changes.

**Use when:**
- **State-dependent behavior**: Behavior depends on state
- **State transitions**: Complex state transitions
- **Avoid conditionals**: Avoid state conditionals

### State Implementation

**State Example:**
```java
// State
public interface State {
    void handle(Context context);
}

public class ConcreteStateA implements State {
    public void handle(Context context) {
        // State A behavior
        context.setState(new ConcreteStateB());
    }
}

public class ConcreteStateB implements State {
    public void handle(Context context) {
        // State B behavior
        context.setState(new ConcreteStateA());
    }
}

// Context
public class Context {
    private State state;
    
    public Context(State state) {
        this.state = state;
    }
    
    public void setState(State state) {
        this.state = state;
    }
    
    public void request() {
        state.handle(this);
    }
}
```

---

## Template Method Pattern

### What is Template Method Pattern?

**Template Method Pattern**: Define algorithm skeleton in base class.

**Use when:**
- **Algorithm structure**: Common algorithm structure
- **Steps vary**: Some steps vary
- **Code reuse**: Reuse algorithm structure

### Template Method Implementation

**Template Method Example:**
```java
// Abstract class
public abstract class DataProcessor {
    // Template method
    public final void process() {
        readData();
        processData();
        saveData();
    }
    
    protected abstract void readData();
    protected abstract void processData();
    
    protected void saveData() {
        System.out.println("Saving data");
    }
}

// Concrete class
public class CSVDataProcessor extends DataProcessor {
    protected void readData() {
        System.out.println("Reading CSV data");
    }
    
    protected void processData() {
        System.out.println("Processing CSV data");
    }
}
```

---

## Best Practices

### 1. Choose Appropriate Pattern

**Why:**
- **Problem fit**: Pattern fits problem
- **Maintainability**: Maintainable code
- **Simplicity**: Keep it simple

**Guidelines:**
- **Understand problem**: Understand the problem
- **Match pattern**: Match pattern to problem
- **Don't force**: Don't force patterns

### 2. Keep Patterns Simple

**Why:**
- **Readability**: Better readability
- **Maintainability**: Easier maintenance
- **Understanding**: Easier understanding

**Guidelines:**
- **Simple implementation**: Keep simple
- **Clear intent**: Clear pattern intent
- **Documentation**: Document usage

### 3. Consider Modern Alternatives

**Why:**
- **Language features**: Modern language features
- **Frameworks**: Framework support
- **Simplicity**: Simpler solutions

**Guidelines:**
- **Language features**: Use language features
- **Frameworks**: Use framework features
- **Modern approaches**: Consider modern approaches

---

## Summary

Behavioral design patterns provide flexible ways to organize object communication and algorithms. Understanding observer, strategy, command, chain of responsibility, state, and template method patterns is essential for effective software design.

**Key Takeaways:**
- **Behavioral patterns**: Design patterns for object communication
- **Observer pattern**: Define one-to-many dependency (notifications, event handling)
- **Strategy pattern**: Define family of algorithms (runtime selection, avoid conditionals)
- **Command pattern**: Encapsulate request as object (queuing, undo/redo)
- **Chain of responsibility**: Pass request along chain of handlers
- **State pattern**: Alter behavior when state changes
- **Template method pattern**: Define algorithm skeleton in base class
- **Best practices**: Choose appropriate pattern, keep simple, consider modern alternatives

**Behavioral Patterns:**
- **Observer**: One-to-many dependency
- **Strategy**: Interchangeable algorithms
- **Command**: Encapsulated requests
- **Chain of Responsibility**: Handler chain
- **State**: State-dependent behavior
- **Template Method**: Algorithm skeleton

**Best Practices:**
- Choose appropriate pattern
- Keep patterns simple
- Consider modern alternatives

**Next Steps:**
- Understand behavioral patterns
- Practice implementation
- Apply patterns appropriately
- Consider modern alternatives

