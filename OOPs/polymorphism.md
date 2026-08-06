# 05 — Polymorphism Deep Dive

## One Interface, Many Forms

---

### What is Polymorphism?

Polymorphism means "many forms." It allows objects of different classes to be treated as objects of a common superclass. The same method call can behave differently depending on the actual object type.

**This is the most asked OOP concept in interviews.**

---

### Two Types of Polymorphism

#### 1. Compile-time (Static) Polymorphism

Resolved at compile time. Achieved through **method overloading**.

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
    
    public String add(String a, String b) {
        return a + b;
    }
}

// Compiler decides which method to call based on arguments
Calculator calc = new Calculator();
calc.add(1, 2);           // Calls add(int, int)
calc.add(1.5, 2.5);       // Calls add(double, double)
calc.add(1, 2, 3);        // Calls add(int, int, int)
calc.add("Hello", " ");   // Calls add(String, String)
```

**Rules for overloading:**
- Same method name
- Different parameter list (number, type, or order)
- Return type alone is NOT enough to distinguish

#### 2. Runtime (Dynamic) Polymorphism

Resolved at runtime. Achieved through **method overriding**.

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

// The actual method called depends on the real object type
Animal animal1 = new Dog();
Animal animal2 = new Cat();

animal1.makeSound();  // "Woof!" — not "Some generic sound"
animal2.makeSound();  // "Meow!"
```

**Key insight:** The reference type is `Animal`, but the actual object determines which `makeSound()` runs. This is **dynamic dispatch**.

---

### How Dynamic Dispatch Works

```
Animal animal = new Dog();
animal.makeSound();

1. Compiler checks: Does Animal have makeSound()? Yes.
2. At runtime: What's the actual object type? Dog.
3. JVM calls Dog.makeSound(), not Animal.makeSound().
```

This is why it's called "runtime polymorphism"—the decision happens at runtime.

---

### The Power of Polymorphism

```java
// WITHOUT polymorphism (bad)
public void processAnimal(Dog dog) { dog.bark(); }
public void processAnimal(Cat cat) { cat.meow(); }
// Must write separate method for each animal type!
// Adding a new animal requires modifying this code!

// WITH polymorphism (good)
public void processAnimal(Animal animal) {
    animal.makeSound();  // Works for ANY animal
}
// Adding a new animal doesn't require changing this code!
```

---

### Polymorphism with Collections

```java
// Store different types in one collection
List<Shape> shapes = new ArrayList<>();
shapes.add(new Circle(5));
shapes.add(new Rectangle(3, 4));
shapes.add(new Triangle(3, 4, 5));

// Process all shapes uniformly
for (Shape shape : shapes) {
    System.out.println("Area: " + shape.area());  // Each computes differently
    shape.draw();  // Each draws differently
}
```

---

### Polymorphism in Real Design

```java
// The Strategy Pattern uses polymorphism
public interface SortStrategy {
    void sort(int[] array);
}

public class BubbleSort implements SortStrategy {
    public void sort(int[] array) { /* bubble sort */ }
}

public class QuickSort implements SortStrategy {
    public void sort(int[] array) { /* quick sort */ }
}

public class MergeSort implements SortStrategy {
    public void sort(int[] array) { /* merge sort */ }
}

// Client code is decoupled from specific algorithm
public class Sorter {
    private SortStrategy strategy;
    
    public Sorter(SortStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sort(int[] array) {
        strategy.sort(array);  // Polymorphism!
    }
    
    // Can change strategy at runtime!
    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }
}
```

---

### instanceof and Downcasting

Sometimes you need to know the actual type:

```java
public void handleShape(Shape shape) {
    shape.draw();  // Polymorphic call
    
    // If you need type-specific behavior
    if (shape instanceof Circle) {
        Circle circle = (Circle) shape;  // Downcast
        System.out.println("Radius: " + circle.getRadius());
    } else if (shape instanceof Rectangle) {
        Rectangle rect = (Rectangle) shape;
        System.out.println("Width: " + rect.getWidth());
    }
}
```

**Warning:** Overusing `instanceof` is a code smell. It often means your design needs polymorphism instead.

```java
// WRONG: Checking type defeats the purpose of polymorphism
if (animal instanceof Dog) {
    ((Dog) animal).bark();
} else if (animal instanceof Cat) {
    ((Cat) animal).meow();
}

// RIGHT: Let polymorphism handle it
animal.makeSound();
```

---

### Covariant Return Types

A subclass can return a more specific type when overriding:

```java
public class Animal {
    public Animal create() {
        return new Animal();
    }
}

public class Dog extends Animal {
    @Override
    public Dog create() {  // Returns Dog, not Animal
        return new Dog();
    }
}
```

---

### Polymorphism in Python

```python
class Animal:
    def make_sound(self):
        print("Some generic sound")

class Dog(Animal):
    def make_sound(self):
        print("Woof!")

class Cat(Animal):
    def make_sound(self):
        print("Meow!")

def process_animal(animal):
    animal.make_sound()  # Polymorphism

process_animal(Dog())  # "Woof!"
process_animal(Cat())  # "Meow!"
```

**Python is duck-typed:** If it has the method, it works—no need for a common parent.

```python
class Car:
    def make_sound(self):
        print("Vroom!")

process_animal(Car())  # "Vroom!" — works even though Car isn't an Animal
```

---

### Common Mistakes

#### Mistake 1: Confusing Reference Type with Object Type

```java
Animal animal = new Dog();
animal.bark();  // Compilation error! Reference type is Animal

// RIGHT: Cast to access Dog-specific method
((Dog) animal).bark();

// Or better: use polymorphism
animal.makeSound();  // Calls Dog's makeSound()
```

#### Mistake 2: Using instanceof Instead of Polymorphism

```java
// WRONG
public void process(Shape shape) {
    if (shape instanceof Circle) {
        // Circle-specific logic
    } else if (shape instanceof Rectangle) {
        // Rectangle-specific logic
    }
}

// RIGHT: Move logic into the classes
public void process(Shape shape) {
    shape.process();  // Each shape handles itself
}
```

#### Mistake 3: Not Understanding Static vs Dynamic Binding

```java
public class Parent {
    public static void staticMethod() {
        System.out.println("Parent static");
    }
    
    public void instanceMethod() {
        System.out.println("Parent instance");
    }
}

public class Child extends Parent {
    public static void staticMethod() {
        System.out.println("Child static");
    }
    
    @Override
    public void instanceMethod() {
        System.out.println("Child instance");
    }
}

Parent p = new Child();
p.staticMethod();    // "Parent static" — static methods use REFERENCE type
p.instanceMethod();  // "Child instance" — instance methods use OBJECT type
```

**Key rule:** Static methods are resolved at compile time (reference type). Instance methods are resolved at runtime (object type).

---

### Interview Questions

**Q: What is polymorphism?**

A: "Polymorphism means 'many forms.' It allows objects of different classes to be treated as objects of a common superclass. The same method call produces different behavior depending on the actual object type. There are two types: compile-time (method overloading) and runtime (method overriding)."

**Q: Explain the difference between overloading and overriding.**

A: "Overloading is multiple methods with the same name but different parameters in the same class—it's resolved at compile time based on the arguments. Overriding is a subclass providing its own implementation of a superclass method—it's resolved at runtime based on the actual object type. Overloading is static polymorphism; overriding is dynamic polymorphism."

**Q: How does dynamic dispatch work?**

A: "When you call a method on a reference, the JVM looks at the actual object type (not the reference type) and calls the appropriate implementation. The compiler verifies the method exists on the reference type, but the JVM decides which implementation to invoke at runtime. This is why `Animal a = new Dog(); a.makeSound()` calls Dog's makeSound()."

**Q: What's the diamond problem and how does Java handle it?**

A: "The diamond problem occurs with multiple inheritance when two parent classes have the same method—the compiler doesn't know which to call. Java avoids this by not allowing multiple class inheritance. Instead, Java uses interfaces, which provide multiple inheritance of type. If two interfaces have the same method, the implementing class provides one implementation, resolving the ambiguity."

---

*Next: [06 — Composition vs Inheritance](06-Composition-vs-Inheritance.md)*
