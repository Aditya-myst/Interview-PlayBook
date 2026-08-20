# 01 — The Four Pillars of OOP

## The Foundation Every Interview Question Builds On

---

### What is Object-Oriented Programming?

OOP is a programming paradigm that organizes code around **objects** rather than functions. Objects bundle data (attributes) and behavior (methods) together, modeling real-world entities.

**Why it matters:** OOP manages complexity. A 10,000-line procedural codebase becomes unmaintainable. OOP breaks it into logical, reusable, testable pieces.

---

### The Four Pillars

```
Object-Oriented Programming
├── 1. Encapsulation    → Hide complexity, protect data
├── 2. Abstraction      → Show only what's necessary
├── 3. Inheritance      → Reuse code through hierarchy
└── 4. Polymorphism     → One interface, many forms
```

---

### Pillar 1: Encapsulation

**Definition:** Bundling data and methods that operate on that data into a single unit (class), and restricting direct access to the internal state.

**Real-world analogy:** A car. You interact with the steering wheel, pedals, and dashboard. You don't access the engine internals directly. The car **encapsulates** the complexity.

```java
// WITHOUT Encapsulation (bad)
public class BankAccount {
    public double balance;  // Anyone can modify this!
}

// With this, anyone can do: account.balance = -1000000;

// WITH Encapsulation (good)
public class BankAccount {
    private double balance;
    
    public double getBalance() {
        return balance;
    }
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public boolean withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }
}

// Now balance is protected. Invalid operations are prevented.
```

**Interview answer:** "Encapsulation means hiding the internal state and requiring all interaction through well-defined methods. This protects data integrity and reduces coupling."

---

### Pillar 2: Abstraction

**Definition:** Exposing only the essential features while hiding the implementation details.

**Real-world analogy:** A TV remote. You press "Power" and "Volume Up." You don't know (or care) about infrared signals, circuit boards, or signal processing. The remote **abstracts** the complexity.

```java
// Abstraction through abstract class
public abstract class Shape {
    abstract double area();        // WHAT (contract)
    abstract double perimeter();   // WHAT (contract)
    
    // Concrete method (shared implementation)
    public void printInfo() {
        System.out.println("Area: " + area());
        System.out.println("Perimeter: " + perimeter());
    }
}

public class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    double area() {
        return Math.PI * radius * radius;  // HOW (implementation)
    }
    
    @Override
    double perimeter() {
        return 2 * Math.PI * radius;       // HOW (implementation)
    }
}
```

**Key difference from Encapsulation:**
- Encapsulation: **hides data** (private fields)
- Abstraction: **hides complexity** (abstract methods, interfaces)

**Interview answer:** "Abstraction focuses on showing only the relevant details to the user. Encapsulation is about bundling data and methods together and restricting access."

---

### Pillar 3: Inheritance

**Definition:** A mechanism where a new class (child/subclass) acquires the properties and behaviors of an existing class (parent/superclass).

**Real-world analogy:** A Dog **is an** Animal. A Car **is a** Vehicle. The child class inherits common behavior and adds specialized behavior.

```java
// Parent class
public class Animal {
    protected String name;
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}

// Child class
public class Dog extends Animal {
    public Dog(String name) {
        this.name = name;
    }
    
    // Dog-specific behavior
    public void bark() {
        System.out.println(name + " is barking");
    }
}

// Usage
Dog dog = new Dog("Buddy");
dog.eat();    // Inherited from Animal
dog.sleep();  // Inherited from Animal
dog.bark();   // Dog-specific
```

**Types of Inheritance:**

| Type | Java | Python | C++ |
|------|------|--------|-----|
| Single | ✓ | ✓ | ✓ |
| Multilevel | ✓ | ✓ | ✓ |
| Hierarchical | ✓ | ✓ | ✓ |
| Multiple | ✗ (use interfaces) | ✓ | ✓ |
| Hybrid | ✗ | ✓ | ✓ |

**Interview warning:** "Java doesn't support multiple inheritance of classes to avoid the diamond problem. It uses interfaces instead."

---

### Pillar 4: Polymorphism

**Definition:** The ability of objects to take many forms. The same method name can behave differently depending on the object.

**Real-world analogy:** The word "draw." A Painter draws a picture. A Bank teller draws money. A Gamer draws a weapon. Same word, different behavior based on context.

**Two types:**

#### Compile-time Polymorphism (Method Overloading)

```java
public class Calculator {
    // Same method name, different parameters
    public int add(int a, int b) {
        return a + b;
    }
    
    public double add(double a, double b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
}

// The compiler decides which method to call based on arguments
```

#### Runtime Polymorphism (Method Overriding)

```java
public class Animal {
    public void makeSound() {
        System.out.println("Some generic sound");
    }
}

public class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}

public class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

// Runtime decides which method to call based on actual object type
Animal animal = new Dog();
animal.makeSound();  // Output: "Woof!" (not "Some generic sound")
```

**Interview answer:** "Polymorphism allows objects of different classes to be treated as objects of a common superclass. The actual method that gets called is determined at runtime based on the object's actual type."

---

### How the Four Pillars Work Together

```java
// Encapsulation: private fields, public methods
// Abstraction: abstract class defines interface
// Inheritance: Dog and Cat extend Animal
// Polymorphism: makeSound() behaves differently

public abstract class Animal {           // Abstraction
    private String name;                 // Encapsulation
    
    public abstract void makeSound();    // Polymorphism contract
    
    public void setName(String name) {   // Encapsulation
        this.name = name;
    }
}

public class Dog extends Animal {        // Inheritance
    @Override
    public void makeSound() {            // Polymorphism
        System.out.println("Woof!");
    }
}

// Client code doesn't need to know the specific type
Animal animal = new Dog();               // Polymorphism
animal.makeSound();                      // Works for any Animal
```

---

### The "Is-A" vs "Has-A" Test

Before using inheritance, ask:

```
Dog IS-A Animal?     → Yes → Inheritance
Car HAS-A Engine?    → Yes → Composition
```

| Relationship | Mechanism | Example |
|--------------|-----------|---------|
| IS-A | Inheritance | Dog extends Animal |
| HAS-A | Composition | Car has Engine |

**This distinction is critical for interviews.** Chapter 6 dives deep into when to use which.

---

### Interview Questions for This Chapter

**Q: What are the four pillars of OOP?**

A: "Encapsulation, Abstraction, Inheritance, and Polymorphism. Encapsulation bundles data and methods and restricts access. Abstraction hides complexity. Inheritance allows code reuse through hierarchy. Polymorphism lets objects take many forms."

**Q: What's the difference between abstraction and encapsulation?**

A: "Abstraction focuses on *what* an object does—it hides complexity by exposing only essential features. Encapsulation focuses on *how*—it bundles data with methods and restricts direct access to the internal state. Abstraction is about design, encapsulation is about implementation."

**Q: Why doesn't Java support multiple inheritance?**

A: "To avoid the diamond problem. If two parent classes have the same method, the compiler wouldn't know which one to call. Java uses interfaces instead, which provide multiple inheritance of type (contract) without the ambiguity."

**Q: What's the difference between overloading and overriding?**

A: "Overloading is compile-time polymorphism—same method name, different parameters in the same class. Overriding is runtime polymorphism—subclass provides its own implementation of a parent method. Overloading is resolved at compile time; overriding is resolved at runtime."

---

### Common Mistakes Freshers Make

1. **Confusing abstraction with encapsulation.** They're related but different.

2. **Overusing inheritance.** "I need to reuse code, so I'll use inheritance." Often, composition is better.

3. **Not understanding runtime polymorphism.** The key insight: the *actual object type* determines which method runs, not the *reference type*.

4. **Memorizing without examples.** Always have a real-world analogy and a code example ready.

---

*Next: [02 — Encapsulation Deep Dive](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/OOPs/encapsulation.md)*
