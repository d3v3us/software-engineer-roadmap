# Software Design Patterns - Structural Deep Dive - Complete Understanding

## Table of Contents
1. [What are Structural Patterns?](#what-are-structural-patterns)
2. [Why Structural Patterns Matter](#why-structural-patterns-matter)
3. [Adapter Pattern](#adapter-pattern)
4. [Decorator Pattern](#decorator-pattern)
5. [Facade Pattern](#facade-pattern)
6. [Proxy Pattern](#proxy-pattern)
7. [Composite Pattern](#composite-pattern)
8. [Bridge Pattern](#bridge-pattern)
9. [Flyweight Pattern](#flyweight-pattern)
10. [Best Practices](#best-practices)

---

## What are Structural Patterns?

### Definition

**Structural Patterns**: Design patterns for composing objects and classes.

**Key Concepts:**
- **Composition**: Compose objects
- **Structure**: Object structure
- **Relationships**: Define relationships
- **Flexibility**: Structural flexibility

### Real-World Analogy

**Structural Patterns = Building Architecture:**
- **Building**: Software system
- **Architecture**: Structural patterns
- **Components**: System components
- **Connections**: Component connections

**Software:**
- **System**: Software system
- **Patterns**: Structural patterns
- **Objects**: System objects
- **Relationships**: Object relationships

---

## Why Structural Patterns Matter?

### Impact of Structure

**1. Flexibility:**
```
Flexible structure
  ↓
Easy changes
  ↓
Adaptability
```

**2. Reusability:**
```
Reusable components
  ↓
Code reuse
  ↓
Efficiency
```

**3. Maintainability:**
```
Clear structure
  ↓
Easy maintenance
  ↓
Maintainable code
```

### Benefits of Structural Patterns

**1. Flexibility:**
- **Structural flexibility**: Flexible object structure
- **Easy changes**: Easy to change structure
- **Extensibility**: Easy to extend

**2. Reusability:**
- **Component reuse**: Reuse components
- **Code reuse**: Code reusability
- **Efficiency**: More efficient

**3. Maintainability:**
- **Clear structure**: Clear code structure
- **Easy maintenance**: Easier maintenance
- **Understanding**: Better understanding

---

## Adapter Pattern

### What is Adapter Pattern?

**Adapter Pattern**: Allow incompatible interfaces to work together.

**Use when:**
- **Interface mismatch**: Incompatible interfaces
- **Legacy code**: Integrate legacy code
- **Third-party**: Integrate third-party libraries

### Adapter Implementation

**Adapter Example:**
```java
// Target interface
public interface MediaPlayer {
    void play(String audioType, String fileName);
}

// Adaptee
public class AdvancedMediaPlayer {
    public void playVlc(String fileName) {
        // Play VLC
    }
    
    public void playMp4(String fileName) {
        // Play MP4
    }
}

// Adapter
public class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;
    
    public MediaAdapter(String audioType) {
        if (audioType.equals("vlc")) {
            advancedPlayer = new VlcPlayer();
        } else if (audioType.equals("mp4")) {
            advancedPlayer = new Mp4Player();
        }
    }
    
    public void play(String audioType, String fileName) {
        if (audioType.equals("vlc")) {
            advancedPlayer.playVlc(fileName);
        } else if (audioType.equals("mp4")) {
            advancedPlayer.playMp4(fileName);
        }
    }
}
```

### Adapter Benefits

**1. Compatibility:**
```
Incompatible interfaces
  ↓
Adapter
  ↓
Compatible
```

**2. Integration:**
```
Legacy code
  ↓
Adapter
  ↓
Integration
```

---

## Decorator Pattern

### What is Decorator Pattern?

**Decorator Pattern**: Add behavior to objects dynamically.

**Use when:**
- **Dynamic behavior**: Add behavior dynamically
- **Flexible composition**: Flexible composition
- **Avoid subclassing**: Avoid subclass explosion

### Decorator Implementation

**Decorator Example:**
```java
// Component
public interface Coffee {
    String getDescription();
    double getCost();
}

// Concrete component
public class SimpleCoffee implements Coffee {
    public String getDescription() {
        return "Simple coffee";
    }
    
    public double getCost() {
        return 1.0;
    }
}

// Decorator
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    public String getDescription() {
        return coffee.getDescription();
    }
    
    public double getCost() {
        return coffee.getCost();
    }
}

// Concrete decorators
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
    
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
}
```

### Decorator Benefits

**1. Flexibility:**
```
Dynamic behavior
  ↓
Flexible composition
  ↓
Runtime composition
```

**2. Extensibility:**
```
Easy to extend
  ↓
New decorators
  ↓
Open/Closed Principle
```

---

## Facade Pattern

### What is Facade Pattern?

**Facade Pattern**: Provide simplified interface to complex subsystem.

**Use when:**
- **Complex subsystem**: Complex subsystem
- **Simplified interface**: Need simplified interface
- **Decoupling**: Decouple client from subsystem

### Facade Implementation

**Facade Example:**
```java
// Complex subsystem
public class CPU {
    public void freeze() { }
    public void jump(long position) { }
    public void execute() { }
}

public class Memory {
    public void load(long position, byte[] data) { }
}

public class HardDrive {
    public byte[] read(long lba, int size) { }
}

// Facade
public class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    
    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    public void startComputer() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}
```

### Facade Benefits

**1. Simplification:**
```
Complex subsystem
  ↓
Simple interface
  ↓
Easy to use
```

**2. Decoupling:**
```
Client decoupled
  ↓
Subsystem changes
  ↓
No client impact
```

---

## Proxy Pattern

### What is Proxy Pattern?

**Proxy Pattern**: Provide surrogate or placeholder for another object.

**Use when:**
- **Lazy loading**: Lazy object loading
- **Access control**: Control access to object
- **Remote access**: Remote object access

### Proxy Implementation

**Proxy Example:**
```java
// Subject
public interface Image {
    void display();
}

// Real subject
public class RealImage implements Image {
    private String fileName;
    
    public RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk(fileName);
    }
    
    public void display() {
        System.out.println("Displaying " + fileName);
    }
    
    private void loadFromDisk(String fileName) {
        System.out.println("Loading " + fileName);
    }
}

// Proxy
public class ProxyImage implements Image {
    private RealImage realImage;
    private String fileName;
    
    public ProxyImage(String fileName) {
        this.fileName = fileName;
    }
    
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}
```

### Proxy Benefits

**1. Lazy Loading:**
```
Lazy loading
  ↓
On-demand creation
  ↓
Performance
```

**2. Access Control:**
```
Access control
  ↓
Security
  ↓
Protection
```

---

## Composite Pattern

### What is Composite Pattern?

**Composite Pattern**: Compose objects into tree structures.

**Use when:**
- **Tree structure**: Tree-like structure
- **Part-whole**: Part-whole hierarchy
- **Uniform treatment**: Treat individual and composite uniformly

### Composite Implementation

**Composite Example:**
```java
// Component
public interface FileSystemNode {
    void display(String indent);
}

// Leaf
public class File implements FileSystemNode {
    private String name;
    
    public File(String name) {
        this.name = name;
    }
    
    public void display(String indent) {
        System.out.println(indent + name);
    }
}

// Composite
public class Directory implements FileSystemNode {
    private String name;
    private List<FileSystemNode> children;
    
    public Directory(String name) {
        this.name = name;
        this.children = new ArrayList<>();
    }
    
    public void add(FileSystemNode node) {
        children.add(node);
    }
    
    public void display(String indent) {
        System.out.println(indent + name + "/");
        for (FileSystemNode child : children) {
            child.display(indent + "  ");
        }
    }
}
```

### Composite Benefits

**1. Uniform Treatment:**
```
Individual and composite
  ↓
Uniform interface
  ↓
Simplified code
```

**2. Flexibility:**
```
Tree structure
  ↓
Flexible composition
  ↓
Easy to extend
```

---

## Bridge Pattern

### What is Bridge Pattern?

**Bridge Pattern**: Separate abstraction from implementation.

**Use when:**
- **Abstraction/Implementation**: Separate abstraction and implementation
- **Runtime binding**: Runtime binding
- **Avoid inheritance**: Avoid inheritance explosion

### Bridge Implementation

**Bridge Example:**
```java
// Implementation
public interface DrawingAPI {
    void drawCircle(double x, double y, double radius);
}

public class DrawingAPI1 implements DrawingAPI {
    public void drawCircle(double x, double y, double radius) {
        System.out.println("API1.circle at " + x + "," + y + " radius " + radius);
    }
}

// Abstraction
public abstract class Shape {
    protected DrawingAPI drawingAPI;
    
    protected Shape(DrawingAPI drawingAPI) {
        this.drawingAPI = drawingAPI;
    }
    
    public abstract void draw();
}

public class Circle extends Shape {
    private double x, y, radius;
    
    public Circle(double x, double y, double radius, DrawingAPI drawingAPI) {
        super(drawingAPI);
        this.x = x;
        this.y = y;
        this.radius = radius;
    }
    
    public void draw() {
        drawingAPI.drawCircle(x, y, radius);
    }
}
```

---

## Flyweight Pattern

### What is Flyweight Pattern?

**Flyweight Pattern**: Share objects to reduce memory usage.

**Use when:**
- **Many objects**: Many similar objects
- **Memory optimization**: Memory optimization
- **Shared state**: Objects with shared state

### Flyweight Implementation

**Flyweight Example:**
```java
// Flyweight
public class TreeType {
    private String name;
    private String color;
    
    public TreeType(String name, String color) {
        this.name = name;
        this.color = color;
    }
    
    public void draw(int x, int y) {
        System.out.println("Drawing " + name + " tree at " + x + "," + y);
    }
}

// Flyweight factory
public class TreeTypeFactory {
    private static Map<String, TreeType> treeTypes = new HashMap<>();
    
    public static TreeType getTreeType(String name, String color) {
        String key = name + color;
        TreeType type = treeTypes.get(key);
        if (type == null) {
            type = new TreeType(name, color);
            treeTypes.put(key, type);
        }
        return type;
    }
}
```

---

## Best Practices

### 1. Choose Appropriate Pattern

**Why:**
- **Right tool**: Use right pattern
- **Problem fit**: Pattern fits problem
- **Maintainability**: Maintainable code

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
- **Simple implementation**: Keep implementation simple
- **Clear intent**: Clear pattern intent
- **Documentation**: Document pattern usage

### 3. Consider Modern Alternatives

**Why:**
- **Modern frameworks**: Modern framework support
- **Language features**: Language features
- **Simplicity**: Simpler solutions

**Guidelines:**
- **Dependency injection**: Consider dependency injection
- **Frameworks**: Use framework features
- **Modern approaches**: Consider modern approaches

---

## Summary

Structural design patterns provide flexible ways to compose objects. Understanding adapter, decorator, facade, proxy, composite, bridge, and flyweight patterns is essential for effective software design.

**Key Takeaways:**
- **Structural patterns**: Design patterns for composing objects
- **Adapter pattern**: Allow incompatible interfaces to work together
- **Decorator pattern**: Add behavior to objects dynamically
- **Facade pattern**: Provide simplified interface to complex subsystem
- **Proxy pattern**: Provide surrogate for another object (lazy loading, access control)
- **Composite pattern**: Compose objects into tree structures
- **Bridge pattern**: Separate abstraction from implementation
- **Flyweight pattern**: Share objects to reduce memory usage
- **Best practices**: Choose appropriate pattern, keep simple, consider modern alternatives

**Structural Patterns:**
- **Adapter**: Interface compatibility
- **Decorator**: Dynamic behavior
- **Facade**: Simplified interface
- **Proxy**: Surrogate object
- **Composite**: Tree structure
- **Bridge**: Abstraction/implementation separation
- **Flyweight**: Object sharing

**Best Practices:**
- Choose appropriate pattern
- Keep patterns simple
- Consider modern alternatives

**Next Steps:**
- Understand structural patterns
- Practice implementation
- Apply patterns appropriately
- Consider modern alternatives

