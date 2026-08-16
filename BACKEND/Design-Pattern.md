# 15 — Design Patterns

## Creational, Structural, Behavioral — 30+ Interview Questions

---

### Singleton

```javascript
class Database {
    constructor() {
        if (Database.instance) return Database.instance;
        this.connection = this.createConnection();
        Database.instance = this;
    }
}
```

---

### Factory

```javascript
class PaymentProcessorFactory {
    static create(type) {
        switch (type) {
            case 'stripe': return new StripeProcessor();
            case 'paypal': return new PayPalProcessor();
            default: throw new Error(`Unknown type: ${type}`);
        }
    }
}
```

---

### Strategy

```javascript
const strategies = {
    bubble: (arr) => { /* bubble sort */ },
    quick: (arr) => { /* quick sort */ },
};

class Sorter {
    constructor(strategy) { this.strategy = strategy; }
    sort(arr) { return this.strategy(arr); }
}
```

---

### Observer

```javascript
class EventEmitter {
    constructor() { this.listeners = {}; }
    on(event, callback) {
        if (!this.listeners[event]) this.listeners[event] = [];
        this.listeners[event].push(callback);
    }
    emit(event, data) {
        this.listeners[event]?.forEach(cb => cb(data));
    }
}
```

---

### Repository Pattern

```javascript
class UserRepository {
    constructor(model) { this.model = model; }
    async findById(id) { return this.model.findById(id); }
    async create(data) { return this.model.create(data); }
    async update(id, data) { return this.model.findByIdAndUpdate(id, data, { new: true }); }
    async delete(id) { return this.model.findByIdAndDelete(id); }
}
```

---

### Interview Questions (30+)

**Q1: What design patterns do you use regularly?**
A: "Singleton for database connections, Factory for creating objects, Strategy for swappable algorithms, Observer for events, Repository for data access, Middleware for request pipeline."

**Q2: What is Singleton pattern?**
A: "Ensure only one instance of class exists. Use for database connections, configuration. Implementation: static instance, private constructor."

**Q3: When would you use Factory pattern?**
A: "Create objects without specifying exact class. When object type determined at runtime. Payment processors, notification senders."

**Q4: What is Strategy pattern?**
A: "Define family of algorithms, make them interchangeable. Use for sorting, payment methods, validation strategies. Client selects algorithm."

**Q5: What is Observer pattern?**
A: "One-to-many dependency. When subject changes, all observers notified. Use for event systems, UI updates, pub/sub."

**Q6: What is Repository pattern?**
A: "Abstract data access layer. Hide database details from business logic. Improve testability. Interface for data operations."

**Q7: What is Dependency Injection?**
A: "Provide dependencies from outside class. Improves testability, flexibility. Constructor injection, setter injection, interface injection."

**Q8: What is Middleware pattern?**
A: "Functions in request-response pipeline. Each middleware processes request/response. Express.js uses this pattern. Order matters."

**Q9: What is Decorator pattern?**
A: "Add behavior to object dynamically. Wrap object with decorator. Use for logging, caching, validation. Similar to middleware."

**Q10: What is Adapter pattern?**
A: "Convert interface of class to another expected by client. Use for integrating third-party libraries. Wraps existing interface."

**Q11: What is Proxy pattern?**
A: "Provide surrogate for another object. Use for lazy loading, access control, caching. Virtual proxy, protection proxy."

**Q12: What is Command pattern?**
A: "Encapsulate request as object. Queue requests, log, support undo. Use for task queues, undo functionality."

**Q13: What is Template Method pattern?**
A: "Define algorithm skeleton in base class. Subclasses implement specific steps. Use for data processing pipelines."

**Q14: What is Builder pattern?**
A: "Construct complex objects step by step. Fluent interface. Use for objects with many optional parameters. HTTP request builder."

**Q15: What is Abstract Factory pattern?**
A: "Create families of related objects. Use when system must be independent of object creation. Cross-platform UI components."

**Q16: What is Prototype pattern?**
A: "Create new objects by cloning existing. Use when creation is expensive. JavaScript uses prototypal inheritance."

**Q17: What is Composite pattern?**
A: "Compose objects into tree structures. Treat individual and composite objects uniformly. File system, UI components."

**Q18: What is Facade pattern?**
A: "Provide simplified interface to complex subsystem. Use for hiding complexity. API gateway is a facade."

**Q19: What is Flyweight pattern?**
A: "Share common state between many objects. Use for memory optimization. Character rendering, connection pooling."

**Q20: What is Chain of Responsibility pattern?**
A: "Pass request along chain of handlers. Each handler decides to process or pass. Express middleware is this pattern."

**Q21: What is Mediator pattern?**
A: "Define object that encapsulates how objects interact. Reduces direct dependencies. Chat room, air traffic control."

**Q22: What is State pattern?**
A: "Object behavior changes when state changes. State objects encapsulate behavior. Use for finite state machines."

**Q23: What is Iterator pattern?**
A: "Access elements sequentially without exposing underlying representation. JavaScript iterators, generators."

**Q24: What is MVC pattern?**
A: "Model-View-Controller. Separates concerns. Model: data. View: presentation. Controller: logic. Web frameworks use this."

**Q25: What is MVVM pattern?**
A: "Model-View-ViewModel. Data binding between view and viewmodel. Use in Angular, Vue. Two-way data binding."

**Q26: What is CQRS pattern?**
A: "Command Query Responsibility Segregation. Separate read and write models. Optimize each independently. Use with event sourcing."

**Q27: What is Event Sourcing pattern?**
A: "Store events instead of current state. Replay events to rebuild state. Audit trail. Use with CQRS."

**Q28: What is Circuit Breaker pattern?**
A: "Prevent cascading failures. States: closed, open, half-open. Trip on error threshold. Fallback response."

**Q29: What is Saga pattern?**
A: "Distributed transaction with compensating actions. Orchestration or choreography. Handle failures gracefully."

**Q30: What is Repository vs Active Record?**
A: "Repository: separate data access from domain. Active Record: model knows how to persist. Repository more flexible; Active Record simpler."

---

### Complete Design Pattern Implementations

```javascript
// Singleton Pattern
class DatabaseConnection {
    constructor() {
        if (DatabaseConnection.instance) {
            return DatabaseConnection.instance;
        }
        this.pool = new Pool({
            host: process.env.DB_HOST,
            port: process.env.DB_PORT,
            database: process.env.DB_NAME,
            user: process.env.DB_USER,
            password: process.env.DB_PASSWORD,
            max: 20
        });
        DatabaseConnection.instance = this;
    }
    
    async query(text, params) {
        return this.pool.query(text, params);
    }
    
    async close() {
        await this.pool.end();
    }
}

// Factory Pattern
class NotificationFactory {
    static create(type) {
        switch (type) {
            case 'email': return new EmailNotification();
            case 'sms': return new SMSNotification();
            case 'push': return new PushNotification();
            default: throw new Error(`Unknown notification type: ${type}`);
        }
    }
}

// Strategy Pattern
const authStrategies = {
    jwt: new JWTStrategy(),
    oauth: new OAuthStrategy(),
    apikey: new APIKeyStrategy()
};

class AuthService {
    constructor(strategy) {
        this.strategy = strategy;
    }
    
    authenticate(req) {
        return this.strategy.authenticate(req);
    }
    
    setStrategy(strategy) {
        this.strategy = strategy;
    }
}

// Observer Pattern
class EventBus {
    constructor() {
        this.listeners = new Map();
    }
    
    on(event, callback) {
        if (!this.listeners.has(event)) {
            this.listeners.set(event, []);
        }
        this.listeners.get(event).push(callback);
    }
    
    off(event, callback) {
        if (this.listeners.has(event)) {
            const callbacks = this.listeners.get(event);
            const index = callbacks.indexOf(callback);
            if (index > -1) {
                callbacks.splice(index, 1);
            }
        }
    }
    
    emit(event, data) {
        if (this.listeners.has(event)) {
            this.listeners.get(event).forEach(cb => cb(data));
        }
    }
}

// Repository Pattern
class UserRepository {
    constructor(prisma) {
        this.prisma = prisma;
    }
    
    async findById(id) {
        return this.prisma.user.findUnique({ where: { id } });
    }
    
    async findByEmail(email) {
        return this.prisma.user.findUnique({ where: { email } });
    }
    
    async create(data) {
        return this.prisma.user.create({ data });
    }
    
    async update(id, data) {
        return this.prisma.user.update({ where: { id }, data });
    }
    
    async delete(id) {
        return this.prisma.user.delete({ where: { id } });
    }
}

// Builder Pattern
class QueryBuilder {
    constructor(table) {
        this.table = table;
        this.conditions = [];
        this.orderByClause = '';
        this.limitClause = '';
        this.offsetClause = '';
    }
    
    where(condition) {
        this.conditions.push(condition);
        return this;
    }
    
    orderBy(column, direction = 'ASC') {
        this.orderByClause = `ORDER BY ${column} ${direction}`;
        return this;
    }
    
    limit(n) {
        this.limitClause = `LIMIT ${n}`;
        return this;
    }
    
    offset(n) {
        this.offsetClause = `OFFSET ${n}`;
        return this;
    }
    
    build() {
        const where = this.conditions.length ? `WHERE ${this.conditions.join(' AND ')}` : '';
        return `SELECT * FROM ${this.table} ${where} ${this.orderByClause} ${this.limitClause} ${this.offsetClause}`.trim();
    }
}

// Usage
const query = new QueryBuilder('users')
    .where('status = active')
    .where('age >= 18')
    .orderBy('name')
    .limit(10)
    .build();
```

---

### Additional Interview Questions (20+)

**Q31: How do you implement Singleton in Node.js?**
A: "Use module caching. Node.js caches modules after first require. Export single instance. Or use class with static instance."

**Q32: What is Dependency Injection benefits?**
A: "Testability (mock dependencies). Flexibility (swap implementations). Loose coupling. Single responsibility."

**Q33: How do you implement Factory pattern?**
A: "Static method that creates objects based on input. Use for creating different types. Payment processors, notification senders."

**Q34: What is Strategy pattern use cases?**
A: "Sorting algorithms, payment methods, authentication strategies, validation rules, compression algorithms."

**Q35: How do you implement Observer pattern?**
A: "Event emitter. Subscribe/unsubscribe. Notify all observers. Use for event systems, pub/sub, UI updates."

**Q36: What is Repository pattern benefits?**
A: "Abstract data access. Testability (mock repository). Switch databases easily. Single responsibility."

**Q37: How do you implement Builder pattern?**
A: "Fluent interface. Chain methods. Build complex objects step by step. Use for objects with many optional parameters."

**Q38: What is Middleware pattern?**
A: "Functions in request-response pipeline. Each middleware processes request/response. Express.js uses this pattern. Order matters."

**Q39: How do you implement Decorator pattern?**
A: "Wrap object with decorator. Add behavior dynamically. Use for logging, caching, validation. Similar to middleware."

**Q40: What is Adapter pattern?**
A: "Convert interface of class to another expected by client. Use for integrating third-party libraries. Wraps existing interface."

**Q41: How do you implement Proxy pattern?**
A: "Provide surrogate for another object. Use for lazy loading, access control, caching. Virtual proxy, protection proxy."

**Q42: What is Command pattern?**
A: "Encapsulate request as object. Queue requests, log, support undo. Use for task queues, undo functionality."

**Q43: How do you implement Template Method pattern?**
A: "Define algorithm skeleton in base class. Subclasses implement specific steps. Use for data processing pipelines."

**Q44: What is Abstract Factory pattern?**
A: "Create families of related objects. Use when system must be independent of object creation. Cross-platform UI components."

**Q45: How do you implement Composite pattern?**
A: "Compose objects into tree structures. Treat individual and composite objects uniformly. File system, UI components."

---

*Next: [16 — Backend Interview Q&A](16-Interview-QA.md)*
