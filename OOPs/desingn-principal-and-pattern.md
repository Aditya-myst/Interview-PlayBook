# 09 — Design Principles & Patterns

## The Patterns That Solve Recurring Design Problems

---

### Beyond SOLID: Essential Design Principles

#### 1. DRY (Don't Repeat Yourself)

```java
// WRONG: Duplicate logic
public class OrderService {
    public double calculateTax(double amount) {
        return amount * 0.1;  // Tax logic duplicated
    }
}

public class InvoiceService {
    public double calculateTax(double amount) {
        return amount * 0.1;  // Same logic!
    }
}

// RIGHT: Extract shared logic
public class TaxCalculator {
    public static double calculate(double amount) {
        return amount * 0.1;
    }
}
```

#### 2. KISS (Keep It Simple, Stupid)

```java
// WRONG: Over-engineered
public interface Validator {
    ValidationResult validate(Object obj);
}
public class CompositeValidator implements Validator { /* ... */ }
public class ValidatorChain { /* ... */ }
// For a simple validation!

// RIGHT: Keep it simple
public boolean isValid(String email) {
    return email != null && email.contains("@");
}
```

#### 3. YAGNI (You Aren't Gonna Need It)

```java
// WRONG: Building for hypothetical future needs
public class User {
    private String name;
    private String email;
    private Address homeAddress;
    private Address workAddress;
    private List<SocialMediaProfile> profiles;
    private Map<String, String> preferences;
    // Do you actually need all this?
}

// RIGHT: Build for current requirements
public class User {
    private String name;
    private String email;
    // Add more when actually needed
}
```

#### 4. Composition Over Inheritance

Already covered in Chapter 6. The key: prefer HAS-A over IS-A.

#### 5. Program to Interface, Not Implementation

```java
// WRONG
ArrayList<String> list = new ArrayList<>();

// RIGHT
List<String> list = new ArrayList<>();
```

#### 6. Encapsulate What Varies

```java
// Identify what changes and encapsulate it
public interface PaymentMethod {
    void pay(double amount);
}

// Payment methods vary—encapsulate them
public class CreditCard implements PaymentMethod { /* ... */ }
public class PayPal implements PaymentMethod { /* ... */ }
public class Crypto implements PaymentMethod { /* ... */ }
```

---

### The 5 Design Patterns You Must Know

#### 1. Singleton

**Problem:** Ensure only one instance of a class exists.

**Use case:** Database connection pool, logger, configuration manager.

```java
public class DatabaseConnection {
    // Lazy initialization (thread-safe)
    private static volatile DatabaseConnection instance;
    
    private DatabaseConnection() {
        // Private constructor prevents external instantiation
    }
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}

// Usage
DatabaseConnection db = DatabaseConnection.getInstance();
```

**Interview answer:** "Singleton ensures a class has exactly one instance. I'd use it for resources that should be shared—like a database connection pool or logger. The double-checked locking pattern ensures thread safety."

#### 2. Factory

**Problem:** Create objects without specifying the exact class.

**Use case:** When you need to create different types based on input.

```java
// Interface
public interface Notification {
    void send(String message);
}

// Implementations
public class EmailNotification implements Notification {
    public void send(String message) { /* send email */ }
}

public class SMSNotification implements Notification {
    public void send(String message) { /* send SMS */ }
}

public class PushNotification implements Notification {
    public void send(String message) { /* send push */ }
}

// Factory
public class NotificationFactory {
    public static Notification create(String type) {
        switch (type.toLowerCase()) {
            case "email": return new EmailNotification();
            case "sms": return new SMSNotification();
            case "push": return new PushNotification();
            default: throw new IllegalArgumentException("Unknown type: " + type);
        }
    }
}

// Usage
Notification notification = NotificationFactory.create("email");
notification.send("Hello!");
```

**Why it matters:** Client code doesn't know about concrete classes. Adding a new type doesn't change the client.

#### 3. Observer

**Problem:** Notify multiple objects when state changes.

**Use case:** Event systems, UI updates, pub/sub.

```java
// Observer interface
public interface Observer {
    void update(String event);
}

// Subject (observable)
public class EventEmitter {
    private Map<String, List<Observer>> listeners = new HashMap<>();
    
    public void on(String event, Observer observer) {
        listeners.computeIfAbsent(event, k -> new ArrayList<>()).add(observer);
    }
    
    public void emit(String event, String data) {
        List<Observer> observers = listeners.getOrDefault(event, Collections.emptyList());
        for (Observer observer : observers) {
            observer.update(data);
        }
    }
}

// Usage
EventEmitter emitter = new EventEmitter();
emitter.on("userCreated", new EmailService());
emitter.on("userCreated", new AnalyticsService());
emitter.emit("userCreated", "user123");  // Both notified
```

#### 4. Strategy

**Problem:** Select algorithm at runtime.

**Use case:** Sorting strategies, payment methods, compression algorithms.

```java
// Strategy interface
public interface SortStrategy {
    void sort(int[] array);
}

// Strategies
public class BubbleSort implements SortStrategy {
    public void sort(int[] array) { /* bubble sort */ }
}

public class QuickSort implements SortStrategy {
    public void sort(int[] array) { /* quick sort */ }
}

public class MergeSort implements SortStrategy {
    public void sort(int[] array) { /* merge sort */ }
}

// Context
public class Sorter {
    private SortStrategy strategy;
    
    public Sorter(SortStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sort(int[] array) {
        strategy.sort(array);
    }
    
    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }
}

// Usage
Sorter sorter = new Sorter(new QuickSort());
sorter.sort(myArray);

// Change strategy at runtime
sorter.setStrategy(new MergeSort());
sorter.sort(myArray);
```

#### 5. Builder

**Problem:** Construct complex objects step by step.

**Use case:** Objects with many optional parameters.

```java
public class HttpRequest {
    private String url;
    private String method;
    private Map<String, String> headers;
    private String body;
    private int timeout;
    
    private HttpRequest(Builder builder) {
        this.url = builder.url;
        this.method = builder.method;
        this.headers = builder.headers;
        this.body = builder.body;
        this.timeout = builder.timeout;
    }
    
    public static class Builder {
        private String url;
        private String method = "GET";
        private Map<String, String> headers = new HashMap<>();
        private String body;
        private int timeout = 30000;
        
        public Builder(String url) {
            this.url = url;
        }
        
        public Builder method(String method) {
            this.method = method;
            return this;
        }
        
        public Builder header(String key, String value) {
            this.headers.put(key, value);
            return this;
        }
        
        public Builder body(String body) {
            this.body = body;
            return this;
        }
        
        public Builder timeout(int timeout) {
            this.timeout = timeout;
            return this;
        }
        
        public HttpRequest build() {
            return new HttpRequest(this);
        }
    }
}

// Usage: Clean, readable
HttpRequest request = new HttpRequest.Builder("https://api.example.com")
    .method("POST")
    .header("Content-Type", "application/json")
    .body("{\"name\": \"John\"}")
    .timeout(5000)
    .build();
```

---

### Patterns Cheat Sheet

| Pattern | Problem | Solution |
|---------|---------|----------|
| Singleton | Need exactly one instance | Private constructor + static getInstance() |
| Factory | Create objects without knowing exact class | Factory method returns interface |
| Strategy | Swap algorithms at runtime | Interface + composition |
| Observer | Notify multiple objects on state change | Event emitter + listeners |
| Builder | Construct complex objects step by step | Fluent builder chain |
| Decorator | Add behavior dynamically | Wrap object with decorator |
| Adapter | Make incompatible interfaces work | Wrapper that converts |
| Template Method | Define algorithm skeleton, let subclasses fill | Abstract class with hooks |

---

### Interview Questions

**Q: What's the Singleton pattern and when would you use it?**

A: "Singleton ensures a class has exactly one instance. I'd use it for resources that should be shared—like a database connection pool, logger, or configuration manager. The double-checked locking pattern ensures thread safety while avoiding unnecessary synchronization."

**Q: Explain the Strategy pattern with an example.**

A: "Strategy lets you swap algorithms at runtime. For example, a payment system might have CreditCardStrategy, PayPalStrategy, and CryptoStrategy. The PaymentProcessor doesn't care which strategy it uses—it just calls pay(). This follows the Open/Closed principle: adding a new payment method doesn't require changing PaymentProcessor."

**Q: What's the difference between Factory and Builder?**

A: "Factory creates different types of objects based on input—it's about WHAT to create. Builder constructs a complex object step by step—it's about HOW to create. Use Factory when you have multiple classes implementing the same interface. Use Builder when one class has many optional parameters."

**Q: When would you use the Observer pattern?**

A: "When you need to notify multiple objects when state changes—like UI components updating when data changes, or event-driven systems. It decouples the subject from its observers, so you can add new observers without modifying the subject."

---

*Next: [10 — OOP System Design](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/OOPs/oops-system-design.md)*
