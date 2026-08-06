# 04 — Inheritance Deep Dive

## Reuse Code Through Hierarchy (But Use It Carefully)

---

### What is Inheritance?

Inheritance allows a new class (child/subclass/derived) to **acquire the properties and methods** of an existing class (parent/superclass/base). The child class can:
- Use parent's methods as-is
- Override parent's methods
- Add new methods and fields

**The keyword:** `extends` (Java), no keyword needed (Python), `:` (C++)

---

### The "Is-A" Test

Before using inheritance, apply this test:

```
Dog IS-A Animal?           → Yes → Inheritance makes sense
Car IS-A Vehicle?          → Yes → Inheritance makes sense
Employee IS-A Company?     → No  → Use composition (Employee HAS-A Company)
```

---

### Basic Inheritance

```java
// Parent class
public class Vehicle {
    protected String brand;
    protected int year;
    
    public Vehicle(String brand, int year) {
        this.brand = brand;
        this.year = year;
    }
    
    public void start() {
        System.out.println(brand + " is starting...");
    }
    
    public void stop() {
        System.out.println(brand + " is stopping...");
    }
    
    public String getInfo() {
        return year + " " + brand;
    }
}

// Child class
public class Car extends Vehicle {
    private int numDoors;
    
    public Car(String brand, int year, int numDoors) {
        super(brand, year);  // Call parent constructor
        this.numDoors = numDoors;
    }
    
    // Override parent method
    @Override
    public String getInfo() {
        return super.getInfo() + " with " + numDoors + " doors";
    }
    
    // New method specific to Car
    public void honk() {
        System.out.println("Beep beep!");
    }
}

// Usage
Car car = new Car("Toyota", 2024, 4);
car.start();     // Inherited from Vehicle
car.stop();      // Inherited from Vehicle
car.honk();      // Car-specific
car.getInfo();   // Overridden in Car
```

---

### Method Overriding

When a child class provides its own implementation of a parent's method.

```java
public class Animal {
    public void makeSound() {
        System.out.println("Some generic animal sound");
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
```

**Rules for overriding:**
1. Same method signature (name + parameters)
2. Same or covariant return type
3. Access modifier can be more visible, not less
4. `@Override` annotation is recommended (catches errors at compile time)

---

### The `super` Keyword

Used to access parent class members.

```java
public class Child extends Parent {
    public Child() {
        super();  // Calls parent constructor (must be first line)
    }
    
    @Override
    public void method() {
        super.method();  // Call parent's version
        // Add child-specific behavior
    }
}
```

---

### Types of Inheritance

#### Single Inheritance
```java
class A { }
class B extends A { }
```

#### Multilevel Inheritance
```java
class Animal { }
class Dog extends Animal { }
class Puppy extends Dog { }
```

#### Hierarchical Inheritance
```java
class Animal { }
class Dog extends Animal { }
class Cat extends Animal { }
```

#### Multiple Inheritance (Not in Java)

Java doesn't support multiple class inheritance. Use interfaces instead:

```java
// WRONG in Java
class Flyable { }
class Swimmable { }
class Duck extends Flyable, Swimmable { }  // Compilation error!

// RIGHT: Use interfaces
interface Flyable { void fly(); }
interface Swimmable { void swim(); }
class Duck implements Flyable, Swimmable {
    public void fly() { }
    public void swim() { }
}
```

**Why? The Diamond Problem:**
```java
class A {
    void method() { System.out.println("A"); }
}

class B extends A {
    void method() { System.out.println("B"); }
}

class C extends A {
    void method() { System.out.println("C"); }
}

// If Java allowed this, which method() would D use?
class D extends B, C { }  // B's or C's? Ambiguity!
```

---

### When NOT to Use Inheritance

#### Anti-Pattern: Inheritance for Code Reuse Only

```java
// WRONG: Stack IS-NOT-A ArrayList
// Stack doesn't need get(index), add(index, element), etc.
public class Stack extends ArrayList<Integer> {
    public void push(int item) { add(item); }
    public int pop() { return remove(size() - 1); }
}

// Problem: Client can do this:
Stack stack = new Stack();
stack.add(0, 999);  // Violates stack's LIFO contract!

// RIGHT: Use composition
public class Stack {
    private List<Integer> items = new ArrayList<>();
    
    public void push(int item) { items.add(item); }
    public int pop() { return items.remove(items.size() - 1); }
    // No add(index, element) method exposed!
}
```

**The Fragile Base Class Problem:**

If the parent class changes its implementation, all child classes can break.

```java
public class Parent {
    public void method() {
        // V1: Does nothing
    }
}

public class Child extends Parent {
    @Override
    public void method() {
        super.method();  // Assumes parent does nothing
        doSomethingImportant();
    }
}

// If Parent.method() later adds behavior, Child might break!
```

---

### Composition vs Inheritance: The Decision

```
Need to reuse code?
    ↓
Is it a true "is-a" relationship?
    ↓
Yes → Inheritance
No  → Composition

Need to change behavior at runtime?
    ↓
Yes → Composition (can swap components)
No  → Inheritance is okay

Need multiple capabilities?
    ↓
Yes → Interfaces + Composition
```

**Interview golden rule:** "Favor composition over inheritance." (From the Gang of Four design patterns book)

---

### Code Examples (4 Languages)

#### Java

```java
public class Animal {
    protected String name;
    public void eat() { System.out.println(name + " eats"); }
}

public class Dog extends Animal {
    public Dog(String name) { this.name = name; }
    public void bark() { System.out.println(name + " barks"); }
}
```

#### Python

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def eat(self):
        print(f"{self.name} eats")

class Dog(Animal):
    def bark(self):
        print(f"{self.name} barks")
```

#### C++

```cpp
class Animal {
protected:
    std::string name;
public:
    virtual void eat() { std::cout << name << " eats" << std::endl; }
    virtual ~Animal() = default;
};

class Dog : public Animal {
public:
    Dog(std::string n) { name = n; }
    void bark() { std::cout << name << " barks" << std::endl; }
};
```

#### JavaScript

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    eat() {
        console.log(`${this.name} eats`);
    }
}

class Dog extends Animal {
    bark() {
        console.log(`${this.name} barks`);
    }
}
```

---

### Common Mistakes

#### Mistake 1: Deep Inheritance Hierarchies

```java
// WRONG: Too deep—hard to understand and maintain
class A { }
class B extends A { }
class C extends B { }
class D extends C { }
class E extends D { }

// RIGHT: Keep hierarchies shallow (2-3 levels max)
// Use composition for additional functionality
```

#### Mistake 2: Using Inheritance for "Has-A" Relationships

```java
// WRONG: Employee IS-NOT-A List<Skill>
class Employee extends ArrayList<Skill> { }

// RIGHT: Employee HAS-A List<Skill>
class Employee {
    private List<Skill> skills = new ArrayList<>();
}
```

#### Mistake 3: Not Calling `super()` Properly

```java
// WRONG: Forgetting to initialize parent
class Child extends Parent {
    public Child(int x) {
        // Parent() constructor not called!
        this.x = x;
    }
}

// RIGHT: Always call super() first
class Child extends Parent {
    public Child(int x) {
        super();  // Initialize parent first
        this.x = x;
    }
}
```

---

### Interview Questions

**Q: When would you use inheritance?**

A: "When there's a true 'is-a' relationship and I want to reuse code or establish a polymorphic hierarchy. For example, `Dog extends Animal` because a Dog is an Animal. But I prefer composition for code reuse without a true hierarchical relationship."

**Q: What's the fragile base class problem?**

A: "When a parent class changes its implementation, child classes can break unexpectedly. This happens because child classes depend on parent's implementation details, not just its interface. This is why 'favor composition over inheritance' is a common guideline."

**Q: Why doesn't Java support multiple inheritance?**

A: "To avoid the diamond problem—ambiguity when two parent classes have the same method. Java uses interfaces instead, which provide multiple inheritance of type without implementation ambiguity."

**Q: What's the difference between overloading and overriding?**

A: "Overloading is multiple methods with the same name but different parameters in the same class—it's resolved at compile time. Overriding is a child class providing its own implementation of a parent method—it's resolved at runtime through dynamic dispatch."

---

*Next: [05 — Polymorphism Deep Dive](05-Polymorphism.md)*
