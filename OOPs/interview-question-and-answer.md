# 11 — Interview Questions & Answers

## Real Questions, Real Answers, Real Confidence

---

### Conceptual Questions

#### Q: What is Object-Oriented Programming?

**A:** "OOP is a programming paradigm that organizes code around objects—bundles of data and behavior. The four pillars are Encapsulation (hiding data), Abstraction (hiding complexity), Inheritance (reusing code through hierarchy), and Polymorphism (one interface, many forms). OOP manages complexity by breaking systems into logical, reusable, testable components."

---

#### Q: What's the difference between abstraction and encapsulation?

**A:** "Abstraction is about *what*—hiding implementation complexity and showing only essential features. It's achieved through abstract classes and interfaces. Encapsulation is about *how*—bundling data with methods and restricting access using private fields. Abstraction simplifies the interface; encapsulation protects the data."

---

#### Q: Explain polymorphism with an example.

**A:** "Polymorphism means 'many forms.' There are two types:

Compile-time: Method overloading—same method name, different parameters. For example, `calculate(int)` and `calculate(double)`.

Runtime: Method overriding—subclass provides its own implementation. For example, `Animal a = new Dog(); a.makeSound()` calls Dog's makeSound(), not Animal's. The actual object type determines which method runs at runtime."

---

#### Q: What's the difference between abstract classes and interfaces?

**A:** "An interface defines a contract—what an object can do. It has only method signatures (and constants in Java 7). An abstract class provides partial implementation—shared code and state. Key differences: a class can implement multiple interfaces but only extend one abstract class. Use interfaces for capabilities (Comparable, Serializable). Use abstract classes for shared implementation in a hierarchy."

---

#### Q: When would you use inheritance vs composition?

**A:** "I use inheritance when there's a true 'is-a' relationship—Dog IS-A Animal. I use composition when it's 'has-a'—Car HAS-A Engine. I prefer composition for code reuse without hierarchy, when I need runtime flexibility, or when I want loose coupling for testability. The guideline is 'favor composition over inheritance' because composition is more flexible and less coupled."

---

### SOLID Questions

#### Q: Explain the SOLID principles with examples.

**A:** "S—Single Responsibility: One class, one job. A UserService handles user logic, not database queries or email sending.

O—Open/Closed: Open for extension, closed for modification. Use polymorphism to add new shapes without changing the calculator.

L—Liskov Substitution: Subtypes must be substitutable. Square shouldn't extend Rectangle because setting width independently violates the contract.

I—Interface Segregation: Don't force clients to depend on unused methods. Split Worker into Workable, Feedable, Sleepable.

D—Dependency Inversion: Depend on abstractions. OrderService depends on Database interface, not MySQLDatabase."

---

#### Q: Give an example of violating and fixing the Liskov Substitution Principle.

**A:** "Classic violation: Square extends Rectangle. Rectangle has independent setWidth() and setHeight(). Square must keep them equal. If client code does rect.setWidth(5); rect.setHeight(10); assert area==50, it fails for Square (area=100).

Fix: Don't use inheritance. Both implement a Shape interface with area(). They're separate classes with no inheritance relationship."

---

#### Q: How does Dependency Inversion help with testing?

**A:** "By depending on interfaces instead of concrete classes, I can inject mocks during testing. OrderService depends on Database interface, not MySQLDatabase. In tests, I inject InMemoryDatabase that doesn't need a real connection. Tests become fast, isolated, and don't depend on external systems."

---

### Design Pattern Questions

#### Q: Explain the Singleton pattern and its issues.

**A:** "Singleton ensures exactly one instance. I'd use it for database connection pools or loggers. Implementation: private constructor, static getInstance() with double-checked locking for thread safety.

Issues: (1) Makes testing hard—can't easily mock a static method. (2) Hidden dependency—classes using Singleton are tightly coupled to it. (3) Violates SRP—manages both its own logic and its lifecycle. Modern approach: use dependency injection instead."

---

#### Q: What's the Strategy pattern and when would you use it?

**A:** "Strategy lets you swap algorithms at runtime. It defines a family of algorithms, encapsulates each one, and makes them interchangeable. Example: payment processing—CreditCardStrategy, PayPalStrategy, CryptoStrategy. The PaymentProcessor doesn't know which strategy it uses; it just calls pay(). This follows Open/Closed: adding a new payment method doesn't require changing PaymentProcessor."

---

#### Q: Explain the Observer pattern.

**A:** "Observer defines a one-to-many dependency. When one object (subject) changes state, all dependents (observers) are notified. Example: EventEmitter emitting 'userCreated' events. EmailService and AnalyticsService subscribe. When a user is created, both are notified automatically. This decouples the subject from its observers."

---

#### Q: What's the Factory pattern?

**A:** "Factory creates objects without specifying the exact class. A NotificationFactory might return EmailNotification or SMSNotification based on a type parameter. The client code works with the Notification interface—it doesn't know about concrete classes. This follows Dependency Inversion and makes adding new types easy."

---

#### Q: What's the Builder pattern and when would you use it?

**A:** "Builder constructs complex objects step by step. It's useful when an object has many optional parameters. Instead of a constructor with 10 parameters, you chain methods: new HttpRequest(url).method('POST').header('Content-Type', 'application/json').body(data).build(). This is readable, avoids telescoping constructors, and ensures the object is fully constructed before use."

---

### Code Design Questions

#### Q: Design a parking lot.

**A:** "Core entities: Vehicle (abstract with Car, Truck, Motorcycle), ParkingSpot, ParkingLevel, ParkingLot, ParkingTicket.

Key design: Vehicle is abstract—each type has getSpotSize(). ParkingSpot tracks availability and vehicle type. ParkingLevel manages a list of spots. ParkingLot has multiple levels and orchestrates parking.

I'd use the Strategy pattern for pricing—different rates for different vehicle types and durations. I'd use composition: ParkingLot HAS ParkingLevels, which HAVE ParkingSpots."

---

#### Q: Design an elevator system.

**A:** "Core entities: Elevator, Floor, Request, ElevatorController.

The Elevator has state (current floor, direction, doors). Requests have floor and direction. The Controller manages multiple elevators and dispatches requests.

I'd use the State pattern for elevator states (MovingUp, MovingDown, Idle). The Strategy pattern for dispatch algorithms (nearest elevator, least busy). The Observer pattern to notify floors when an elevator arrives."

---

#### Q: Design a chess game.

**A:** "Core entities: Piece (abstract), King, Queen, Rook, Bishop, Knight, Pawn, Board, Player, Game.

Each piece type has isValidMove(from, to, board). The Board manages the 8x8 grid and executes moves. The Game manages turns and game state.

Key design: Polymorphism—each piece implements its own move validation. The Board doesn't know about specific piece types; it just asks 'is this move valid?' This follows Open/Closed: adding a new piece type doesn't require changing Board."

---

### Tricky Questions

#### Q: Can an interface have a constructor?

**A:** "No. Interfaces cannot have constructors because they cannot be instantiated directly. Only classes that implement the interface can be instantiated. Abstract classes can have constructors, which are called when a subclass is instantiated."

---

#### Q: Can a constructor be private?

**A:** "Yes. This is used in the Singleton pattern to prevent external instantiation. The class provides a static method (getInstance()) that controls the creation of the single instance."

---

#### Q: What happens if you don't call super() in a child constructor?

**A:** "In Java, if you don't explicitly call super(), the compiler inserts super() (no-arg parent constructor) as the first line. If the parent doesn't have a no-arg constructor, you'll get a compilation error—you must explicitly call super(args) with the appropriate parameters."

---

#### Q: Can an abstract class have a constructor?

**A:** "Yes. Abstract classes can have constructors. They're called when a concrete subclass is instantiated. The constructor initializes the abstract class's fields. You can't instantiate an abstract class directly, but its constructor runs when a child is created."

---

#### Q: What's the difference between == and .equals()?

**A:** "== compares references (do they point to the same object?). .equals() compares content (do they have the same value?). For Strings, 'hello' == 'hello' might be true due to string interning, but you should always use .equals() for content comparison."

---

#### Q: Why is String immutable in Java?

**A:** "Several reasons: (1) Security—Strings are used for class loading, network connections. Immutability prevents malicious modification. (2) Thread safety—immutable objects are inherently thread-safe. (3) String pool—Java can safely share String literals. (4) Hashcode caching—since the value doesn't change, hashCode() can be computed once and cached."

---

### Behavioral Questions (OOP-Related)

#### Q: Tell me about a time you refactored code using OOP principles.

**A:** "I had a UserService class that handled user logic, database queries, email sending, and PDF generation. It was over 500 lines and violated SRP. I refactored it: UserRepository for database, EmailService for emails, PdfGenerator for PDFs. The UserService became a coordinator, delegating to these services. This made each class testable independently and the code much easier to maintain."

---

#### Q: How do you decide which design pattern to use?

**A:** "I identify the problem first: (1) Need one instance? Singleton. (2) Create objects without knowing exact type? Factory. (3) Swap algorithms at runtime? Strategy. (4) Notify multiple objects? Observer. (5) Build complex objects? Builder. I don't force patterns—I use them when they naturally fit. Over-engineering is worse than no pattern."

---

#### Q: When is it okay to violate SOLID?

**A:** "In small scripts or prototypes where maintainability isn't a concern. Or when a violation is contained—like a simple if/else that's unlikely to grow. The principles are guidelines, not laws. But for production code, following SOLID usually pays off in maintainability and testability."

---

### The "Stumped" Strategy

If you're stumped in an interview:

1. **Think out loud.** "Let me think about this... The key question is..."
2. **Start with what you know.** "I know that interfaces define contracts..."
3. **Give an example.** "For example, in a payment system..."
4. **Ask for clarification.** "Do you mean in general or in a specific context?"

---

### The 30-Second OOP Summary

If asked to summarize OOP in 30 seconds:

"OOP organizes code around objects that bundle data and behavior. The four pillars are: Encapsulation—hiding data behind methods. Abstraction—hiding complexity behind simple interfaces. Inheritance—reusing code through hierarchy. Polymorphism—one interface, many forms. SOLID principles guide good design: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion. Design patterns like Strategy, Observer, and Factory solve recurring problems."

---

### Final Tips

1. **Always have examples ready.** Don't just define—illustrate.
2. **Know when NOT to use something.** "I wouldn't use inheritance here because..."
3. **Explain trade-offs.** "The benefit is... the cost is..."
4. **Practice explaining out loud.** OOP questions test communication, not just knowledge.
5. **Read the Gang of Four book.** Even just the introduction and a few patterns.

---

*Good luck with your interviews! You've got this.*
