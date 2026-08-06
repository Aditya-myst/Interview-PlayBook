# 06 — JavaScript Advanced

## Closures, Prototypes, Event Loop, this

---

### Closures

A **closure** is a function that remembers the variables from its outer scope, even after the outer function has returned.

```javascript
function outer() {
    let count = 0;  // This variable is "closed over"
    
    return function inner() {
        count++;
        return count;
    };
}

const counter = outer();
counter();  // 1
counter();  // 2
counter();  // 3

// count is not accessible directly
// console.log(count);  // ReferenceError
```

#### Why Closures Matter

```javascript
// 1. Data privacy (like private variables)
function createBankAccount(balance) {
    return {
        deposit(amount) { balance += amount; },
        withdraw(amount) { balance -= amount; },
        getBalance() { return balance; }
    };
}

const account = createBankAccount(100);
account.deposit(50);
account.getBalance();  // 150
// balance is not directly accessible

// 2. Function factories
function multiply(factor) {
    return (number) => number * factor;
}

const double = multiply(2);
const triple = multiply(3);
double(5);  // 10
triple(5);  // 15

// 3. Event handlers with state
function createButton(name) {
    let clicks = 0;
    return {
        click() {
            clicks++;
            console.log(`${name} clicked ${clicks} times`);
        }
    };
}
```

#### Classic Closure Problem

```javascript
// PROBLEM: var in loops
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
// Prints: 3, 3, 3 (not 0, 1, 2)

// SOLUTION 1: Use let
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
// Prints: 0, 1, 2

// SOLUTION 2: IIFE closure
for (var i = 0; i < 3; i++) {
    (function(j) {
        setTimeout(() => console.log(j), 100);
    })(i);
}
```

---

### Prototypes

Every JavaScript object has a prototype—another object it inherits from.

```javascript
// Prototype chain
const animal = {
    eat() { return 'eating'; }
};

const dog = Object.create(animal);
dog.bark = function() { return 'woof'; };

dog.bark();  // 'woof' (own property)
dog.eat();   // 'eating' (inherited from animal)

// Constructor function
function Person(name, age) {
    this.name = name;
    this.age = age;
}

Person.prototype.greet = function() {
    return `Hi, I'm ${this.name}`;
};

const alice = new Person('Alice', 30);
alice.greet();  // "Hi, I'm Alice"

// ES6 Classes (syntactic sugar over prototypes)
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    greet() {
        return `Hi, I'm ${this.name}`;
    }
    
    static create(name, age) {
        return new Person(name, age);
    }
}
```

---

### The `this` Keyword

`this` depends on how a function is called, not where it's defined.

```javascript
// 1. Global context
console.log(this);  // window (browser) or global (Node)

// 2. Object method
const obj = {
    name: 'Alice',
    greet() {
        console.log(this.name);  // 'Alice'
    }
};

// 3. Standalone function
function sayHello() {
    console.log(this);  // window (non-strict) or undefined (strict)
}

// 4. Arrow function (inherits from parent)
const obj2 = {
    name: 'Alice',
    greet: () => {
        console.log(this.name);  // undefined (this = outer scope)
    }
};

// 5. Explicit binding
function greet() {
    console.log(this.name);
}

const alice = { name: 'Alice' };
greet.call(alice);    // 'Alice'
greet.apply(alice);   // 'Alice'
const bound = greet.bind(alice);
bound();              // 'Alice'

// 6. Event handler
button.addEventListener('click', function() {
    console.log(this);  // The button element
});

button.addEventListener('click', () => {
    console.log(this);  // window (arrow function)
});
```

---

### Event Loop

JavaScript is single-threaded. The event loop handles async operations.

```
┌───────────────────────────┐
│        Call Stack          │  ← Executes synchronous code
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│      Web APIs             │  ← setTimeout, fetch, DOM events
│   (browser/Node provides) │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│    Microtask Queue        │  ← Promises, queueMicrotask
│    (higher priority)       │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│    Macrotask Queue         │  ← setTimeout, setInterval, I/O
│    (lower priority)        │
└───────────────────────────┘
```

```javascript
console.log('1');              // Call stack

setTimeout(() => {
    console.log('2');          // Macrotask
}, 0);

Promise.resolve().then(() => {
    console.log('3');          // Microtask
});

console.log('4');              // Call stack

// Output: 1, 4, 3, 2
// Synchronous first, then microtasks, then macrotasks
```

---

### Event Delegation

Instead of adding event listeners to each child, add one to the parent.

```javascript
// BAD: Adding listener to each item
document.querySelectorAll('.item').forEach(item => {
    item.addEventListener('click', handleClick);
});

// GOOD: Event delegation
document.querySelector('.list').addEventListener('click', (e) => {
    if (e.target.matches('.item')) {
        handleClick(e);
    }
});

// Works with dynamically added elements too!
```

---

### Interview Questions

**Q: What is a closure?**

A: "A function that remembers variables from its outer scope even after the outer function returns. Used for data privacy, function factories, and maintaining state. Example: counter function that increments a private variable."

**Q: Explain the event loop.**

A: "JavaScript is single-threaded. The event loop processes: (1) Call stack (synchronous code), (2) Microtask queue (Promises), (3) Macrotask queue (setTimeout). Microtasks have priority over macrotasks."

**Q: What's the difference between `call`, `apply`, and `bind`?**

A: "call: invokes function with specified `this` and arguments individually. apply: same but arguments as array. bind: returns new function with bound `this` (doesn't invoke immediately)."

**Q: How does `this` work in JavaScript?**

A: "`this` depends on how function is called: method call → object, standalone → window/undefined, arrow function → inherited from parent, constructor → new instance, event handler → element. Use bind/call/apply to explicitly set."

---

*Next: [07 — Async JavaScript](07-Async-JS.md)*
