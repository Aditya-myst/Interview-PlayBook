# 08 — SOLID Principles

## The 5 Principles That Make Software Maintainable

---

### What is SOLID?

SOLID is an acronym for five design principles that make object-oriented software easier to understand, maintain, and extend.

| Letter | Principle | One-Line Summary |
|--------|-----------|------------------|
| **S** | Single Responsibility | One class, one job |
| **O** | Open/Closed | Open for extension, closed for modification |
| **L** | Liskov Substitution | Subtypes must be substitutable |
| **I** | Interface Segregation | Many specific interfaces, one fat one |
| **D** | Dependency Inversion | Depend on abstractions, not concretions |

**Interview tip:** "SOLID principles guide us to write code that's easy to maintain, test, and extend. They reduce coupling and increase cohesion."

---

### S — Single Responsibility Principle (SRP)

**"A class should have only one reason to change."**

Each class should have exactly ONE job.

```java
// WRONG: One class doing everything
public class Employee {
    public void calculatePay() { /* ... */ }
    public void saveToDatabase() { /* ... */ }
    public void generateReport() { /* ... */ }
    public void sendEmail() { /* ... */ }
}
// This class changes if: pay logic changes, DB schema changes,
// report format changes, or email service changes. Too many reasons!

// RIGHT: Separate responsibilities
public class Employee {
    private String name;
    private double salary;
    // Employee data only
}

public class PayCalculator {
    public double calculatePay(Employee emp) { /* ... */ }
}

public class EmployeeRepository {
    public void save(Employee emp) { /* ... */ }
}

public class ReportGenerator {
    public String generateReport(Employee emp) { /* ... */ }
}

public class EmailService {
    public void sendEmail(Employee emp, String message) { /* ... */ }
}
```

**How to check:** "How many reasons does this class have to change?" If more than one, split it.

---

### O — Open/Closed Principle (OCP)

**"Software entities should be open for extension, closed for modification."**

You should be able to add new behavior WITHOUT changing existing code.

```java
// WRONG: Must modify this class for every new shape
public class AreaCalculator {
    public double calculate(Object shape) {
        if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return Math.PI * c.getRadius() * c.getRadius();
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.getWidth() * r.getHeight();
        }
        // Adding Triangle means modifying this method!
        return 0;
    }
}

// RIGHT: Open for extension, closed for modification
public interface Shape {
    double area();
}

public class Circle implements Shape {
    private double radius;
    public double area() { return Math.PI * radius * radius; }
}

public class Rectangle implements Shape {
    private double width, height;
    public double area() { return width * height; }
}

// Adding Triangle doesn't require changing AreaCalculator!
public class Triangle implements Shape {
    private double base, height;
    public double area() { return 0.5 * base * height; }
}

public class AreaCalculator {
    public double calculate(Shape shape) {
        return shape.area();  // Polymorphism handles it!
    }
}
```

**The key:** Use polymorphism and interfaces. New classes extend behavior; existing code stays untouched.

---

### L — Liskov Substitution Principle (LSP)

**"Subtypes must be substitutable for their base types without altering correctness."**

If you replace a parent object with a child object, the program should still work correctly.

```java
// WRONG: Violates LSP
public class Rectangle {
    protected int width, height;
    
    public void setWidth(int w) { width = w; }
    public void setHeight(int h) { height = h; }
    public int area() { return width * height; }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int w) {
        width = w;
        height = w;  // Force square
    }
    
    @Override
    public void setHeight(int h) {
        width = h;  // Force square
        height = h;
    }
}

// This code works for Rectangle but breaks for Square!
public void process(Rectangle rect) {
    rect.setWidth(5);
    rect.setHeight(10);
    assert rect.area() == 50;  // FAILS for Square! (area = 100)
}

// RIGHT: Separate classes, no inheritance
public interface Shape {
    double area();
}

public class Rectangle implements Shape {
    private int width, height;
    public double area() { return width * height; }
}

public class Square implements Shape {
    private int side;
    public double area() { return side * side; }
}
```

**The test:** "Can I replace the parent with any child and the code still works?" If no, LSP is violated.

---

### I — Interface Segregation Principle (ISP)

**"Clients should not be forced to depend on methods they don't use."**

```java
// WRONG: Fat interface
public interface Machine {
    void print();
    void scan();
    void fax();
}

public class SimplePrinter implements Machine {
    public void print() { /* Works */ }
    public void scan() { throw new UnsupportedOperationException(); }
    public void fax() { throw new UnsupportedOperationException(); }
}

// RIGHT: Segregated interfaces
public interface Printable {
    void print();
}

public interface Scannable {
    void scan();
}

public interface Faxable {
    void fax();
}

public class SimplePrinter implements Printable {
    public void print() { /* Works */ }
}

public class AllInOnePrinter implements Printable, Scannable, Faxable {
    public void print() { /* ... */ }
    public void scan() { /* ... */ }
    public void fax() { /* ... */ }
}
```

---

### D — Dependency Inversion Principle (DIP)

**"High-level modules should not depend on low-level modules. Both should depend on abstractions."**

```java
// WRONG: High-level depends on low-level
public class OrderService {
    private MySQLDatabase database;  // Depends on concrete class!
    
    public OrderService() {
        this.database = new MySQLDatabase();  // Tight coupling!
    }
    
    public void saveOrder(Order order) {
        database.save(order);
    }
}

// RIGHT: Both depend on abstraction
public interface Database {
    void save(Object entity);
}

public class OrderService {
    private Database database;  // Depends on interface!
    
    public OrderService(Database database) {  // Injected!
        this.database = database;
    }
    
    public void saveOrder(Order order) {
        database.save(order);
    }
}

// Can easily swap implementations
OrderService service1 = new OrderService(new MySQLDatabase());
OrderService service2 = new OrderService(new PostgreSQLDatabase());
OrderService service3 = new OrderService(new InMemoryDatabase());  // For testing!
```

**Key benefit:** Easy testing! You can inject a mock database without changing OrderService.

---

### SOLID in Action: Complete Example

```java
// BEFORE SOLID
public class Report {
    public String generate() { /* complex logic */ }
    public void saveToDatabase() { /* DB logic */ }
    public void emailTo(String address) { /* email logic */ }
    public void exportToPdf() { /* PDF logic */ }
}

// AFTER SOLID
// S: Single Responsibility
public class Report {
    private String data;
    public String getData() { return data; }
}

public class ReportGenerator {
    public Report generate() { /* ... */ }
}

// O: Open/Closed (use strategy for export)
public interface ReportExporter {
    void export(Report report);
}

public class PdfExporter implements ReportExporter {
    public void export(Report report) { /* PDF logic */ }
}

public class CsvExporter implements ReportExporter {
    public void export(Report report) { /* CSV logic */ }
}

// L: Subtypes are substitutable
// Any ReportExporter works wherever ReportExporter is expected

// I: Segregated interfaces
public interface ReportSavable {
    void save(Report report);
}

public interface ReportEmailable {
    void email(Report report, String address);
}

// D: Depend on abstractions
public class ReportService {
    private ReportGenerator generator;
    private ReportExporter exporter;
    private ReportSavable repository;
    
    public ReportService(ReportGenerator gen, ReportExporter exp, ReportSavable repo) {
        this.generator = gen;
        this.exporter = exp;
        this.repository = repo;
    }
    
    public void processReport() {
        Report report = generator.generate();
        exporter.export(report);
        repository.save(report);
    }
}
```

---

### Common Violations and Fixes

| Violation | Example | Fix |
|-----------|---------|-----|
| SRP | Class handles DB + email + PDF | Split into separate classes |
| OCP | if/else for each type | Use polymorphism |
| LSP | Square extends Rectangle | Separate classes with common interface |
| ISP | Robot implements Worker with eat() | Segregate interfaces |
| DIP | new MySQLDatabase() in constructor | Inject via constructor |

---

### Interview Questions

**Q: What are the SOLID principles ?**

A: "SOLID stands for Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion. They're design principles that make code more maintainable, testable, and extensible. SRP says one class, one job. OCP says extend without modifying. LSP says subtypes must be substitutable. ISP says prefer specific interfaces. DIP says depend on abstractions."

**Q: Give an example of Liskov Substitution Principle violation.**

A: "The classic example is Square extending Rectangle. Rectangle allows setting width and height independently, but Square must keep them equal. If client code sets width=5 and height=10 expecting area=50, it breaks for Square where area=100. The fix is to not use inheritance here—have separate classes implementing a common Shape interface."

**Q: How does Dependency Inversion help with testing ?**

A: "By depending on interfaces instead of concrete classes, I can inject mock implementations during testing. For example, OrderService depends on a Database interface, not MySQLDatabase. In tests, I inject an InMemoryDatabase that doesn't need a real database connection. This makes tests fast and isolated."

**Q: When is it okay to violate SOLID ?**

A: "In small, simple scripts or prototypes where maintainability isn't a concern. Or when the violation doesn't cause real problems—like a simple if/else that's unlikely to grow. The principles are guidelines, not laws. But for production code that will be maintained, following SOLID usually pays off."

---

*Next: [09 — Design Principles & Patterns](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/OOPs/desingn-principal-and-pattern.md)*
