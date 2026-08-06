# 07 — Interfaces & Abstract Classes

## Defining Contracts and Sharing Implementation

---

### The Key Difference

| Aspect | Interface | Abstract Class |
|--------|-----------|----------------|
| **Purpose** | Define a contract (WHAT) | Share implementation (HOW) |
| **Methods** | Abstract only (Java 7) | Abstract + Concrete |
| **Fields** | Constants only | Instance variables |
| **Constructor** | No | Yes |
| **Multiple** | A class can implement many | A class can extend only one |
| **Relationship** | "Can-do" capability | "Is-a" hierarchy |

**Interview answer:** "An interface defines *what* an object can do—it's a contract. An abstract class provides *how* some things are done—it's a partial implementation. Use interfaces for capabilities, abstract classes for shared code in a hierarchy."

---

### Interfaces

An interface is a **pure contract**. It says: "Any class that implements me MUST provide these methods."

```java
public interface Payable {
    double calculatePay();
    String getPaymentMethod();
}

public class Employee implements Payable {
    private double salary;
    
    @Override
    public double calculatePay() {
        return salary;
    }
    
    @Override
    public String getPaymentMethod() {
        return "Direct Deposit";
    }
}

public class Contractor implements Payable {
    private double hourlyRate;
    private int hours;
    
    @Override
    public double calculatePay() {
        return hourlyRate * hours;
    }
    
    @Override
    public String getPaymentMethod() {
        return "Check";
    }
}

// Client code works with the interface
public void processPayment(Payable payable) {
    double amount = payable.calculatePay();
    String method = payable.getPaymentMethod();
    // Process payment...
}
```

**Key benefits:**
- Client code doesn't know if it's an Employee or Contractor
- Can add new Payable types without changing client code
- Enables polymorphism across unrelated classes

---

### Abstract Classes

An abstract class is a **partially implemented class**. It can have both abstract and concrete methods.

```java
public abstract class Shape {
    protected String color;
    
    // Constructor
    public Shape(String color) {
        this.color = color;
    }
    
    // Abstract methods: subclasses MUST implement
    public abstract double area();
    public abstract double perimeter();
    
    // Concrete method: shared implementation
    public String getDescription() {
        return color + " shape with area " + area();
    }
    
    // Template method pattern
    public void printInfo() {
        System.out.println("Color: " + color);
        System.out.println("Area: " + area());
        System.out.println("Perimeter: " + perimeter());
    }
}

public class Circle extends Shape {
    private double radius;
    
    public Circle(String color, double radius) {
        super(color);  // Call abstract class constructor
        this.radius = radius;
    }
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public double perimeter() {
        return 2 * Math.PI * radius;
    }
}
```

---

### When to Use What

#### Use an Interface When:
- You're defining a **capability** (Comparable, Serializable, Drawable)
- **Unrelated classes** need to implement it
- You need **multiple inheritance** of type
- You're defining a **contract** without implementation

```java
// Good interface use: capability
public interface Sortable {
    int compareTo(Sortable other);
}

public class Student implements Sortable { /* ... */ }
public class Product implements Sortable { /* ... */ }
// Student and Product are unrelated but both sortable
```

#### Use an Abstract Class When:
- You have **shared implementation** for subclasses
- Subclasses share **common state** (fields)
- You want to provide **default behavior**
- There's a clear **is-a hierarchy**

```java
// Good abstract class use: shared implementation
public abstract class AbstractRepository<T> {
    protected Connection connection;  // Shared state
    
    public AbstractRepository(Connection conn) {
        this.connection = conn;
    }
    
    // Shared implementation
    public T findById(int id) {
        String sql = buildFindByIdQuery();
        return executeQuery(sql, id);
    }
    
    // Subclasses provide specifics
    protected abstract String buildFindByIdQuery();
    protected abstract T mapRow(ResultSet rs);
}
```

---

### Modern Java: Default Methods in Interfaces

Since Java 8, interfaces can have default implementations:

```java
public interface Logger {
    void log(String message);
    
    // Default method
    default void logError(String message) {
        log("ERROR: " + message);
    }
    
    default void logWarning(String message) {
        log("WARNING: " + message);
    }
}
```

This blurs the line between interfaces and abstract classes, but the key difference remains: a class can implement multiple interfaces but only extend one class.

---

### The Interface Segregation Principle

Don't force clients to depend on methods they don't use.

```java
// WRONG: Fat interface
public interface Worker {
    void work();
    void eat();
    void sleep();
}

// A Robot can work but doesn't eat or sleep!
public class Robot implements Worker {
    public void work() { /* ... */ }
    public void eat() { /* Not applicable! */ }
    public void sleep() { /* Not applicable! */ }
}

// RIGHT: Segregated interfaces
public interface Workable {
    void work();
}

public interface Feedable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

public class HumanWorker implements Workable, Feedable, Sleepable {
    public void work() { /* ... */ }
    public void eat() { /* ... */ }
    public void sleep() { /* ... */ }
}

public class Robot implements Workable {
    public void work() { /* ... */ }
    // No need to implement eat() or sleep()
}
```

---

### Real-World Example: Payment System

```java
// Interface: What can be done
public interface PaymentProcessor {
    PaymentResult process(double amount, Currency currency);
    boolean refund(String transactionId);
}

// Abstract class: Shared implementation
public abstract class AbstractPaymentProcessor implements PaymentProcessor {
    protected String apiKey;
    protected Logger logger;
    
    public AbstractPaymentProcessor(String apiKey) {
        this.apiKey = apiKey;
        this.logger = LoggerFactory.getLogger(getClass());
    }
    
    // Template method: shared workflow
    @Override
    public PaymentResult process(double amount, Currency currency) {
        logger.info("Processing payment: " + amount + " " + currency);
        validate(amount, currency);
        PaymentResult result = doProcess(amount, currency);
        logResult(result);
        return result;
    }
    
    // Shared validation
    protected void validate(double amount, Currency currency) {
        if (amount <= 0) throw new IllegalArgumentException("Invalid amount");
    }
    
    // Subclasses implement the actual processing
    protected abstract PaymentResult doProcess(double amount, Currency currency);
    
    private void logResult(PaymentResult result) {
        logger.info("Payment result: " + result);
    }
}

// Concrete class: Specific implementation
public class StripeProcessor extends AbstractPaymentProcessor {
    public StripeProcessor(String apiKey) {
        super(apiKey);
    }
    
    @Override
    protected PaymentResult doProcess(double amount, Currency currency) {
        // Call Stripe API
        return new PaymentResult(true, "stripe_txn_123");
    }
    
    @Override
    public boolean refund(String transactionId) {
        // Call Stripe refund API
        return true;
    }
}
```

---

### Python: ABC and Protocols

```python
from abc import ABC, abstractmethod

# Abstract class
class Shape(ABC):
    def __init__(self, color):
        self.color = color
    
    @abstractmethod
    def area(self):
        pass
    
    def describe(self):  # Concrete method
        return f"{self.color} shape with area {self.area()}"

# Interface-like (Protocol in Python 3.8+)
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self):  # Satisfies Drawable protocol implicitly
        print("Drawing circle")
```

---

### Common Mistakes

#### Mistake 1: Using Abstract Class When Interface Is Better

```java
// WRONG: No shared implementation
public abstract class Flyable {
    public abstract void fly();  // Why abstract class?
}

// RIGHT: Interface
public interface Flyable {
    void fly();
}
```

#### Mistake 2: Making Everything an Interface

```java
// WRONG: Over-engineering
public interface Nameable { String getName(); }
public interface Ageable { int getAge(); }
public interface Addressable { String getAddress(); }

// If every Person has all three, just put them in one class
public class Person {
    private String name;
    private int age;
    private String address;
    // getters...
}
```

#### Mistake 3: Not Programming to the Interface

```java
// WRONG: Depending on concrete implementation
public class OrderService {
    private MySQLDatabase db;  // Tightly coupled!
}

// RIGHT: Depending on abstraction
public class OrderService {
    private Database db;  // Interface
}
```

---

### Interview Questions

**Q: When would you use an interface vs an abstract class?**

A: "I use an interface when defining a capability that unrelated classes might share—like Comparable or Serializable. I use an abstract class when I have shared implementation in a hierarchy—like a base Repository with common CRUD operations. The key difference: a class can implement multiple interfaces but only extend one abstract class."

**Q: What are default methods in interfaces?**

A: "Since Java 8, interfaces can have default method implementations. This allows adding new methods to interfaces without breaking existing implementations. It's useful for backward compatibility. However, the primary purpose of interfaces remains defining contracts, not providing implementation."

**Q: What is the Interface Segregation Principle?**

A: "Clients shouldn't be forced to depend on methods they don't use. Instead of one fat interface, create multiple specific interfaces. For example, instead of one Worker interface with work(), eat(), and sleep(), create Workable, Feedable, and Sleepable interfaces. A Robot only implements Workable."

**Q: Can an interface have state (fields)?**

A: "In Java, interfaces can only have public static final constants, not instance variables. This is because interfaces define behavior contracts, not state. If you need shared state, use an abstract class."

---

*Next: [08 — SOLID Principles](08-SOLID-Principles.md)*
