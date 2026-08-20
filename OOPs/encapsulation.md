# 02 — Encapsulation Deep Dive

## The Art of Protecting Your Data

---

### What is Encapsulation?

Encapsulation is the practice of **bundling data (fields) and methods that operate on that data into a single unit (class)**, and **restricting direct access** to the internal state.

Think of it as a protective capsule around your data.

---

### Why It Matters

**Without encapsulation:**
```java
public class BankAccount {
    public double balance;
}

BankAccount account = new BankAccount();
account.balance = -1000000;  // Anyone can set invalid state!
account.balance = 0.0001;    // No validation!
```

**With encapsulation:**
```java
public class BankAccount {
    private double balance;
    
    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Amount must be positive");
        balance += amount;
    }
    
    public boolean withdraw(double amount) {
        if (amount <= 0 || amount > balance) return false;
        balance -= amount;
        return true;
    }
    
    public double getBalance() {
        return balance;
    }
}

BankAccount account = new BankAccount();
account.deposit(1000);
account.withdraw(500);
// account.balance = -1000000;  // Compilation error! Can't access directly.
```

---

### The Three Access Modifiers

| Modifier | Same Class | Same Package | Subclass | Everywhere |
|----------|-----------|--------------|----------|------------|
| `public` | ✓ | ✓ | ✓ | ✓ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| (default) | ✓ | ✓ | ✗ | ✗ |
| `private` | ✓ | ✗ | ✗ | ✗ |

**Interview tip:** "I use `private` for fields, `public` for methods that clients need, and `protected` only when subclasses need access."

---

### Getters and Setters

The standard way to control access to private fields.

```java
public class Person {
    private String name;
    private int age;
    
    // Getter
    public String getName() {
        return name;
    }
    
    // Setter with validation
    public void setName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        this.name = name.trim();
    }
    
    // Getter
    public int getAge() {
        return age;
    }
    
    // Setter with validation
    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age: " + age);
        }
        this.age = age;
    }
}
```

**Key point:** Setters are not just assignment—they enforce **invariants** (rules that must always be true).

---

### Immutable Objects

The ultimate form of encapsulation: objects that cannot be changed after creation.

```java
public final class Money {
    private final double amount;
    private final String currency;
    
    public Money(double amount, String currency) {
        if (amount < 0) throw new IllegalArgumentException("Amount cannot be negative");
        this.amount = amount;
        this.currency = currency;
    }
    
    public double getAmount() {
        return amount;
    }
    
    public String getCurrency() {
        return currency;
    }
    
    // No setters! Object is immutable.
    
    // Operations return NEW objects instead of modifying state
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Currency mismatch");
        }
        return new Money(this.amount + other.amount, this.currency);
    }
}
```

**Why immutability is powerful:**
- Thread-safe without synchronization
- Can be shared freely
- No side effects
- Easier to reason about

**Interview answer:** "Immutable objects are the strongest form of encapsulation. Once created, their state cannot change. This makes them thread-safe and easier to reason about."

---

### Data Hiding vs Data Protection

| Concept | Meaning | Example |
|---------|---------|---------|
| Data Hiding | Internal fields not visible | `private` fields |
| Data Protection | Invalid states prevented | Validation in setters |

**Both are part of encapsulation.**

---

### The Benefits of Encapsulation

1. **Data Integrity:** Validation ensures the object is always in a valid state
2. **Flexibility:** Internal implementation can change without affecting clients
3. **Maintainability:** Changes are localized to the class
4. **Testability:** Each class can be tested independently
5. **Security:** Sensitive data is protected from unauthorized access

---

### Real-World Example: User Authentication

```java
public class User {
    private String username;
    private String passwordHash;  // Never store plain passwords!
    private List<String> roles;
    
    public User(String username, String password) {
        this.username = username;
        this.passwordHash = hashPassword(password);
        this.roles = new ArrayList<>();
    }
    
    public boolean checkPassword(String password) {
        return passwordHash.equals(hashPassword(password));
    }
    
    public boolean hasRole(String role) {
        return roles.contains(role);
    }
    
    // Client code doesn't know HOW passwords are stored
    // Implementation can change from SHA-256 to bcrypt without breaking clients
    private String hashPassword(String password) {
        // Implementation detail hidden from outside
        return HashUtils.sha256(password);
    }
    
    // No getter for passwordHash—security!
    public String getUsername() {
        return username;
    }
}
```

**Notice:** There's no `getPasswordHash()` method. Some data shouldn't be exposed at all.

---

### Common Mistakes

#### Mistake 1: Making Everything Public

```java
// WRONG: No encapsulation
public class Student {
    public String name;
    public int grade;
    public List<String> courses;
}

// RIGHT: Controlled access
public class Student {
    private String name;
    private int grade;
    private List<String> courses;
    
    public String getName() { return name; }
    public void setGrade(int grade) {
        if (grade < 0 || grade > 100) throw new IllegalArgumentException();
        this.grade = grade;
    }
    public List<String> getCourses() {
        return Collections.unmodifiableList(courses);  // Return read-only view
    }
}
```

#### Mistake 2: Exposing Mutable Objects

```java
// WRONG: Internal list can be modified from outside
public List<String> getCourses() {
    return courses;  // Caller can add/remove items!
}

// RIGHT: Return unmodifiable copy
public List<String> getCourses() {
    return Collections.unmodifiableList(new ArrayList<>(courses));
}
```

#### Mistake 3: Getters/Setters for Everything

```java
// WRONG: Just making fields private with public getters/setters
// This is NOT real encapsulation—it's just syntactic sugar

// RIGHT: Only expose what clients actually need
public class Temperature {
    private double celsius;
    
    // Maybe clients only need fahrenheit
    public double getFahrenheit() {
        return celsius * 9/5 + 32;
    }
    
    // Don't expose celsius at all if clients don't need it
}
```

---

### Interview Questions

**Q: What is encapsulation and why is it important?**

A: "Encapsulation bundles data and methods together and restricts direct access to internal state. It's important because: (1) it protects data integrity through validation, (2) it allows internal implementation to change without breaking client code, (3) it reduces coupling between components, and (4) it makes code easier to maintain and test."

**Q: What's the difference between data hiding and encapsulation?**

A: "Data hiding is a consequence of encapsulation. Encapsulation is the broader concept of bundling data with methods. Data hiding specifically refers to making internal fields private so they can't be accessed directly. Encapsulation achieves data hiding through access modifiers."

**Q: When would you make a class immutable?**

A: "When the object represents a value that shouldn't change—like Money, Date, or a configuration object. Immutability makes objects thread-safe, prevents bugs from unexpected state changes, and makes code easier to reason about. The trade-off is that you create new objects for every change."

**Q: Should getters and setters always be provided?**

A: "No. Only expose what clients actually need. If a field is purely internal, don't provide a getter. If a field shouldn't change after construction, don't provide a setter. Every public method is a commitment to maintain that interface."

---

*Next: [03 — Abstraction Deep Dive](https://github.com/Aditya-myst/Interview-PlayBook/edit/main/OOPs/Abstraction.md)*
