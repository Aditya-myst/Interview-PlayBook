# 06 — Composition vs Inheritance

## The Decision That Separates Juniors from Seniors

---

### The Golden Rule

**"Favor composition over inheritance."**
— Gang of Four, *Design Patterns* (1994)

This is the single most important design principle that separates junior developers from senior ones.

---

### What is Composition?

Composition means building complex objects by **combining simpler objects** as components. Instead of inheriting behavior, you **contain** it.

**Real-world analogy:** A Car is composed of Engine, Transmission, Wheels, etc. A Car doesn't extend Engine—it has an Engine.

---

### Inheritance: The "Is-A" Relationship

```java
// Inheritance: Dog IS-A Animal
public class Dog extends Animal {
    // Gets all Animal behavior
    // Can override specific methods
}
```

### Composition: The "Has-A" Relationship

```java
// Composition: Car HAS-A Engine
public class Car {
    private Engine engine;  // Car contains an Engine
    
    public Car(Engine engine) {
        this.engine = engine;
    }
    
    public void start() {
        engine.start();  // Delegates to component
    }
}
```

---

### The Classic Mistake

```java
// WRONG: Using inheritance for code reuse
// Stack IS-NOT-A ArrayList, but we want ArrayList's functionality
public class Stack extends ArrayList<Integer> {
    public void push(int item) { add(item); }
    public int pop() { return remove(size() - 1); }
}

// Problem: Client can do this:
Stack stack = new Stack();
stack.add(0, 999);  // Violates LIFO contract!
stack.get(5);       // Doesn't make sense for a stack!

// RIGHT: Composition
public class Stack {
    private List<Integer> items = new ArrayList<>();
    
    public void push(int item) { items.add(item); }
    public int pop() { return items.remove(items.size() - 1); }
    public boolean isEmpty() { return items.isEmpty(); }
    // No add(index, element) or get(index) exposed!
}
```

---

### When to Use Inheritance

Use inheritance when:
1. There's a true **"is-a"** relationship
2. The child class **is a specialized version** of the parent
3. You need **polymorphism** through the hierarchy
4. The parent class is **designed for inheritance** (documented, stable)

```java
// GOOD use of inheritance
public abstract class Shape {
    abstract double area();
    abstract double perimeter();
}

public class Circle extends Shape {
    private double radius;
    
    @Override
    double area() { return Math.PI * radius * radius; }
    
    @Override
    double perimeter() { return 2 * Math.PI * radius; }
}

public class Rectangle extends Shape {
    private double width, height;
    
    @Override
    double area() { return width * height; }
    
    @Override
    double perimeter() { return 2 * (width + height); }
}
```

---

### When to Use Composition

Use composition when:
1. It's a **"has-a"** relationship
2. You need **flexibility** to change behavior at runtime
3. You want to **reuse code** without tight coupling
4. The relationship is **not hierarchical**

```java
// GOOD use of composition
public class Car {
    private Engine engine;
    private Transmission transmission;
    private List<Wheel> wheels;
    
    public Car(Engine engine, Transmission transmission) {
        this.engine = engine;
        this.transmission = transmission;
        this.wheels = Arrays.asList(
            new Wheel(), new Wheel(), new Wheel(), new Wheel()
        );
    }
    
    public void drive() {
        engine.start();
        transmission.shift(1);
        // ...
    }
    
    // Can swap components at runtime!
    public void setEngine(Engine newEngine) {
        this.engine = newEngine;
    }
}
```

---

### The Flexibility Advantage

```java
// Inheritance: Behavior fixed at compile time
public class Bird extends Animal {
    public void move() { fly(); }  // Always flies
}

// Composition: Behavior can change at runtime
public class Bird {
    private MoveStrategy moveStrategy;
    
    public Bird(MoveStrategy strategy) {
        this.moveStrategy = strategy;
    }
    
    public void move() {
        moveStrategy.move();
    }
    
    public void setMoveStrategy(MoveStrategy strategy) {
        this.moveStrategy = strategy;  // Can change at runtime!
    }
}

// Usage
Bird eagle = new Bird(new FlyStrategy());
eagle.move();  // Flies

eagle.setMoveStrategy(new WalkStrategy());
eagle.move();  // Walks (maybe it's injured)
```

---

### The Delegation Pattern

Composition often uses delegation—the containing class forwards calls to its components.

```java
public class EmailService {
    private SmtpClient smtpClient;     // Component
    private TemplateEngine templateEngine;  // Component
    
    public EmailService(SmtpClient smtp, TemplateEngine template) {
        this.smtpClient = smtp;
        this.templateEngine = template;
    }
    
    public void sendEmail(String to, String templateName, Map<String, String> data) {
        // Delegate to template engine
        String body = templateEngine.render(templateName, data);
        
        // Delegate to SMTP client
        smtpClient.send(to, "Subject", body);
    }
}

// EmailService doesn't extend SmtpClient or TemplateEngine
// It HAS-A SmtpClient and HAS-A TemplateEngine
```

---

### Real-World Example: Notification System

```java
// WRONG: Inheritance approach
public class EmailNotification { /* email logic */ }
public class SMSNotification extends EmailNotification { /* override email with SMS */ }
public class PushNotification extends EmailNotification { /* override email with push */ }
// Problem: SMS IS-NOT-A EmailNotification!

// RIGHT: Composition approach
public interface NotificationSender {
    void send(Notification notification);
}

public class EmailSender implements NotificationSender {
    public void send(Notification notification) { /* email logic */ }
}

public class SMSSender implements NotificationSender {
    public void send(Notification notification) { /* SMS logic */ }
}

public class PushSender implements NotificationSender {
    public void send(Notification notification) { /* push logic */ }
}

public class NotificationService {
    private List<NotificationSender> senders;
    
    public NotificationService(List<NotificationSender> senders) {
        this.senders = senders;
    }
    
    public void notifyUser(User user, String message) {
        Notification notification = new Notification(user, message);
        for (NotificationSender sender : senders) {
            sender.send(notification);
        }
    }
}
```

---

### The Decision Matrix

| Factor | Inheritance | Composition |
|--------|------------|-------------|
| Relationship | Is-a | Has-a |
| Coupling | Tight | Loose |
| Flexibility | Fixed at compile time | Can change at runtime |
| Code Reuse | Through hierarchy | Through delegation |
| Testing | Harder (must test parent) | Easier (mock components) |
| Extensibility | Add subclasses | Add/swap components |
| Complexity | Simple for small hierarchies | Scales better |

---

### The Hybrid Approach

In practice, you often use both:

```java
// Abstract base defines common interface
public abstract class AbstractRepository<T> {
    protected abstract Connection getConnection();
    
    public T findById(int id) {
        Connection conn = getConnection();
        // Common query logic using conn
    }
}

// Concrete implementations use composition for details
public class UserRepository extends AbstractRepository<User> {
    private ConnectionPool pool;  // Composition
    
    @Override
    protected Connection getConnection() {
        return pool.getConnection();  // Delegate to pool
    }
}
```

---

### Code Examples (4 Languages)

#### Java

```java
// Composition
public class Computer {
    private CPU cpu;
    private RAM ram;
    private HardDrive hardDrive;
    
    public Computer(CPU cpu, RAM ram, HardDrive hd) {
        this.cpu = cpu;
        this.ram = ram;
        this.hardDrive = hd;
    }
    
    public void boot() {
        cpu.process();
        ram.load();
        hardDrive.read();
    }
}
```

#### Python

```python
# Composition
class Computer:
    def __init__(self, cpu, ram, hard_drive):
        self.cpu = cpu
        self.ram = ram
        self.hard_drive = hard_drive
    
    def boot(self):
        self.cpu.process()
        self.ram.load()
        self.hard_drive.read()
```

#### C++

```cpp
class Computer {
    CPU cpu;
    RAM ram;
    HardDrive hardDrive;
public:
    Computer(CPU c, RAM r, HardDrive h) : cpu(c), ram(r), hardDrive(h) {}
    void boot() { cpu.process(); ram.load(); hardDrive.read(); }
};
```

#### JavaScript

```javascript
class Computer {
    constructor(cpu, ram, hardDrive) {
        this.cpu = cpu;
        this.ram = ram;
        this.hardDrive = hardDrive;
    }
    
    boot() {
        this.cpu.process();
        this.ram.load();
        this.hardDrive.read();
    }
}
```

---

### Common Mistakes

#### Mistake 1: Inheritance for Code Reuse

```java
// WRONG
public class UserService extends DatabaseUtils {
    // Reusing DatabaseUtils methods through inheritance
}

// RIGHT
public class UserService {
    private DatabaseUtils dbUtils;  // Composition
    
    public UserService(DatabaseUtils dbUtils) {
        this.dbUtils = dbUtils;
    }
}
```

#### Mistake 2: Composition for True Hierarchies

```java
// WRONG: Dog IS-A Animal, use inheritance
public class Dog {
    private Animal animal;  // Unnecessary indirection
}

// RIGHT
public class Dog extends Animal { }
```

#### Mistake 3: Exposing Internal Components

```java
// WRONG: Breaks encapsulation
public class Car {
    public Engine engine;  // Should be private!
}

// RIGHT: Controlled access
public class Car {
    private Engine engine;
    
    public void start() { engine.start(); }
    // Don't expose engine directly
}
```

---

### Interview Questions

**Q: When would you choose composition over inheritance?**

A: "I prefer composition when: (1) the relationship is 'has-a' not 'is-a', (2) I need flexibility to change behavior at runtime, (3) I want loose coupling for testability, or (4) the inheritance hierarchy would be deep. The classic example is Stack—instead of extending ArrayList, I compose it internally and only expose stack operations."

**Q: What's the fragile base class problem?**

A: "When a parent class changes its implementation, child classes can break because they depend on parent's internal behavior, not just its interface. With composition, components interact through well-defined interfaces, so changes in one component don't cascade to others."

**Q: Can you give an example where inheritance caused problems?**

A: "Java's Stack class extends Vector. This means you can call Vector methods like add(index, element) on a Stack, violating the LIFO contract. A better design would use composition—internally use a List but only expose push, pop, peek, and isEmpty."

**Q: How do you decide between inheritance and composition in a design?**

A: "I ask: 'Is this a true is-a relationship?' If Cat IS-A Animal, inheritance makes sense. If Car HAS-A Engine, composition. I also consider: will I need to swap behavior at runtime? If yes, composition with Strategy pattern. Is the parent class designed for extension? If not, composition is safer."

---

*Next: [07 — Interfaces & Abstract Classes](07-Interfaces-and-Abstract-Classes.md)*
