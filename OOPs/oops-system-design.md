# 10 — OOP System Design

## Putting It All Together: Design Real Systems

---

### Why System Design Matters for Freshers

Even for fresher roles, top companies ask OOP design questions. They want to see:
1. Can you identify classes and relationships?
2. Can you apply SOLID principles?
3. Can you use design patterns appropriately?
4. Can you think about extensibility?

---

### The System Design Framework

For any OOP design problem, follow these steps:

```
Step 1: Clarify Requirements
  → What does the system do?
  → What are the use cases?
  → What are the constraints?

Step 2: Identify Core Entities
  → What are the main objects?
  → What data do they hold?
  → What behavior do they have?

Step 3: Define Relationships
  → Is-a (inheritance)?
  → Has-a (composition)?
  → Uses-a (dependency)?

Step 4: Apply Design Principles
  → SOLID
  → Appropriate design patterns
  → Encapsulation

Step 5: Code the Core
  → Interfaces first
  → Key classes
  → Show extensibility
```

---

### Design Problem 1: Parking Lot

**Requirements:**
- Multiple levels, each with parking spots
- Different vehicle types (car, truck, motorcycle)
- Track available spots
- Park and unpark vehicles

**Step 1: Identify Entities**

```
Vehicle (abstract)
├── Car
├── Truck
└── Motorcycle

ParkingLot
└── ParkingLevel
    └── ParkingSpot

ParkingTicket
```

**Step 2: Design**

```java
// Enums
public enum VehicleType {
    CAR, TRUCK, MOTORCYCLE
}

// Vehicle hierarchy
public abstract class Vehicle {
    private String licensePlate;
    private VehicleType type;
    
    public Vehicle(String licensePlate, VehicleType type) {
        this.licensePlate = licensePlate;
        this.type = type;
    }
    
    public abstract int getSpotSize();  // How many spots needed
}

public class Car extends Vehicle {
    public Car(String plate) { super(plate, VehicleType.CAR); }
    public int getSpotSize() { return 1; }
}

public class Truck extends Vehicle {
    public Truck(String plate) { super(plate, VehicleType.TRUCK); }
    public int getSpotSize() { return 2; }
}

public class Motorcycle extends Vehicle {
    public Motorcycle(String plate) { super(plate, VehicleType.MOTORCYCLE); }
    public int getSpotSize() { return 1; }
}

// Parking Spot
public class ParkingSpot {
    private int spotNumber;
    private VehicleType type;  // What vehicle can park here
    private Vehicle vehicle;   // Currently parked vehicle (null if empty)
    
    public boolean isAvailable() { return vehicle == null; }
    
    public boolean canFit(Vehicle v) {
        return isAvailable() && v.getType() == type;
    }
    
    public void park(Vehicle v) {
        if (!canFit(v)) throw new IllegalArgumentException("Can't fit");
        this.vehicle = v;
    }
    
    public Vehicle unpark() {
        Vehicle v = this.vehicle;
        this.vehicle = null;
        return v;
    }
}

// Parking Level
public class ParkingLevel {
    private int levelNumber;
    private List<ParkingSpot> spots;
    
    public Optional<ParkingSpot> findAvailableSpot(VehicleType type) {
        return spots.stream()
            .filter(spot -> spot.canFit(new Car("")))  // Check type
            .findFirst();
    }
}

// Parking Lot
public class ParkingLot {
    private List<ParkingLevel> levels;
    
    public ParkingTicket park(Vehicle vehicle) {
        for (ParkingLevel level : levels) {
            Optional<ParkingSpot> spot = level.findAvailableSpot(vehicle.getType());
            if (spot.isPresent()) {
                spot.get().park(vehicle);
                return new ParkingTicket(level.getLevelNumber(), 
                    spot.get().getSpotNumber(), vehicle);
            }
        }
        throw new RuntimeException("Parking lot full");
    }
    
    public double unpark(ParkingTicket ticket) {
        // Calculate fee and unpark
        return ticket.calculateFee();
    }
}
```

---

### Design Problem 2: Library Management System

**Requirements:**
- Books with copies
- Members can borrow and return
- Track due dates
- Search by title/author

```java
// Core entities
public class Book {
    private String isbn;
    private String title;
    private String author;
    private List<BookCopy> copies;
}

public class BookCopy {
    private String copyId;
    private Book book;
    private boolean available;
}

public class Member {
    private String memberId;
    private String name;
    private List<BorrowRecord> borrowHistory;
}

public class BorrowRecord {
    private BookCopy copy;
    private Member member;
    private LocalDate borrowDate;
    private LocalDate dueDate;
    private LocalDate returnDate;
    
    public boolean isOverdue() {
        LocalDate checkDate = returnDate != null ? returnDate : LocalDate.now();
        return checkDate.isAfter(dueDate);
    }
}

// Service classes (SRP)
public class LibraryCatalog {
    private Map<String, Book> books;
    
    public List<Book> searchByTitle(String title) { /* ... */ }
    public List<Book> searchByAuthor(String author) { /* ... */ }
}

public class BorrowingService {
    private LibraryCatalog catalog;
    private int maxBooksPerMember = 5;
    
    public BorrowRecord borrow(Member member, Book book) {
        // Check member can borrow
        // Find available copy
        // Create borrow record
        // Mark copy as unavailable
    }
    
    public void returnBook(BorrowRecord record) {
        // Mark copy as available
        // Set return date
        // Calculate fine if overdue
    }
}
```

---

### Design Problem 3: E-Commerce System

```java
// Product hierarchy
public abstract class Product {
    private String id;
    private String name;
    private double price;
    private int stock;
}

public class PhysicalProduct extends Product {
    private double weight;
    private Dimensions dimensions;
}

public class DigitalProduct extends Product {
    private String downloadUrl;
}

// Shopping
public class CartItem {
    private Product product;
    private int quantity;
    
    public double getSubtotal() {
        return product.getPrice() * quantity;
    }
}

public class ShoppingCart {
    private List<CartItem> items;
    
    public void addItem(Product product, int quantity) { /* ... */ }
    public void removeItem(Product product) { /* ... */ }
    public double getTotal() {
        return items.stream()
            .mapToDouble(CartItem::getSubtotal)
            .sum();
    }
}

// Order processing
public class Order {
    private String orderId;
    private Customer customer;
    private List<CartItem> items;
    private Payment payment;
    private ShippingInfo shipping;
    private OrderStatus status;
}

// Strategy for payment
public interface PaymentStrategy {
    boolean pay(double amount);
}

public class CreditCardPayment implements PaymentStrategy { /* ... */ }
public class PayPalPayment implements PaymentStrategy { /* ... */ }

// Strategy for shipping
public interface ShippingStrategy {
    double calculateCost(double weight);
    int estimateDays();
}

public class StandardShipping implements ShippingStrategy { /* ... */ }
public class ExpressShipping implements ShippingStrategy { /* ... */ }
```

---

### Design Problem 4: ATM Machine

```java
public class ATM {
    private CashDispenser dispenser;
    private BankService bankService;
    private Screen screen;
    
    public void processTransaction(Card card, String pin, TransactionRequest request) {
        // 1. Authenticate
        if (!bankService.authenticate(card, pin)) {
            screen.showError("Invalid PIN");
            return;
        }
        
        // 2. Process based on type
        switch (request.getType()) {
            case WITHDRAWAL:
                processWithdrawal(card, request.getAmount());
                break;
            case BALANCE:
                double balance = bankService.getBalance(card);
                screen.displayBalance(balance);
                break;
            case TRANSFER:
                bankService.transfer(card, request.getTargetAccount(), request.getAmount());
                break;
        }
    }
    
    private void processWithdrawal(Card card, double amount) {
        if (!bankService.hasSufficientFunds(card, amount)) {
            screen.showError("Insufficient funds");
            return;
        }
        if (!dispenser.canDispense(amount)) {
            screen.showError("Cannot dispense this amount");
            return;
        }
        bankService.debit(card, amount);
        dispenser.dispense(amount);
        screen.showSuccess("Please take your cash");
    }
}

public class CashDispenser {
    private Map<Integer, Integer> denominations;  // denomination -> count
    
    public boolean canDispense(double amount) { /* ... */ }
    public void dispense(double amount) { /* ... */ }
}
```

---

### Design Problem 5: Chess Game

```java
public abstract class Piece {
    protected Color color;
    protected Position position;
    
    public abstract boolean isValidMove(Position from, Position to, Board board);
}

public class King extends Piece {
    public boolean isValidMove(Position from, Position to, Board board) {
        // King moves one square in any direction
        int dx = Math.abs(from.getX() - to.getX());
        int dy = Math.abs(from.getY() - to.getY());
        return dx <= 1 && dy <= 1;
    }
}

public class Rook extends Piece {
    public boolean isValidMove(Position from, Position to, Board board) {
        // Rook moves in straight lines
        return from.getX() == to.getX() || from.getY() == to.getY();
    }
}

public class Board {
    private Piece[][] grid = new Piece[8][8];
    
    public boolean movePiece(Position from, Position to) {
        Piece piece = getPieceAt(from);
        if (piece == null) return false;
        if (!piece.isValidMove(from, to, this)) return false;
        
        // Execute move
        grid[to.getX()][to.getY()] = piece;
        grid[from.getX()][from.getY()] = null;
        piece.setPosition(to);
        return true;
    }
}

public class Game {
    private Board board;
    private Player white;
    private Player black;
    private Player currentPlayer;
    
    public void makeMove(Position from, Position to) {
        if (board.movePiece(from, to)) {
            switchPlayer();
            if (isCheckmate()) {
                // Game over
            }
        }
    }
}
```

---

### Common Design Problems for Interviews

| Problem | Key Concepts |
|---------|--------------|
| Parking Lot | Vehicle hierarchy, Strategy for pricing |
| Library | Book/Copy separation, Borrowing service |
| E-Commerce | Product hierarchy, Cart, Order, Payment strategy |
| ATM | State pattern, Strategy for transactions |
| Chess | Piece hierarchy, Board, Move validation |
| Elevator | State pattern, Request queue |
| Hotel Booking | Room hierarchy, Booking service, Pricing strategy |
| Social Media | User, Post, Observer for notifications |
| File System | Composite pattern (files and directories) |
| Snake Game | Board, Snake, Food, Direction enum |

---

### The Interview Approach

1. **Clarify** (2 min): Ask questions about requirements
2. **Identify** (3 min): List core entities and relationships
3. **Design** (10 min): Draw class diagram, define interfaces
4. **Code** (10 min): Implement key classes
5. **Discuss** (5 min): Extensibility, SOLID, patterns used

---

*Next: [11 — Interview Questions & Answers](11-Interview-QA.md)*
