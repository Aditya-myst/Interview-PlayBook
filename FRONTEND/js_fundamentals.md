# 05 — JavaScript Fundamentals

## ES6+, Types, Scope, Hoisting

---

### var vs let vs const

```javascript
// var: function-scoped, hoisted, can be redeclared
var x = 1;
var x = 2;  // OK
console.log(x);  // 2

// let: block-scoped, not hoisted (temporal dead zone)
let y = 1;
// let y = 2;  // Error: already declared
y = 2;  // OK

// const: block-scoped, must be initialized, can't be reassigned
const z = 1;
// z = 2;  // Error: assignment to constant

// But const objects can be mutated
const obj = { name: 'Alice' };
obj.name = 'Bob';  // OK (mutating, not reassigning)
// obj = {};  // Error: reassigning
```

| Feature | var | let | const |
|---------|-----|-----|-------|
| Scope | Function | Block | Block |
| Hoisting | Yes (undefined) | TDZ | TDZ |
| Redeclaration | Yes | No | No |
| Reassignment | Yes | Yes | No |

---

### Data Types

```javascript
// Primitive types (immutable, passed by value)
const str = "hello";        // String
const num = 42;             // Number
const big = 9007199254740991n; // BigInt
const bool = true;          // Boolean
const undef = undefined;    // Undefined
const empty = null;         // Null
const sym = Symbol('id');   // Symbol

// Reference types (mutable, passed by reference)
const arr = [1, 2, 3];      // Array
const obj = { name: 'Alice' }; // Object
const fn = function() {};    // Function
const date = new Date();     // Date
const map = new Map();       // Map
const set = new Set();       // Set
```

---

### Type Coercion

```javascript
// Implicit coercion
"5" + 3      // "53" (string concatenation)
"5" - 3      // 2 (numeric subtraction)
true + 1     // 2
false + ""   // "false"

// Explicit coercion
Number("5")      // 5
String(5)        // "5"
Boolean(0)       // false
Boolean("")      // false
Boolean(null)    // false
Boolean(undefined) // false
Boolean(NaN)     // false
Boolean(1)       // true
Boolean("hello") // true

// == vs ===
5 == "5"     // true (type coercion)
5 === "5"    // false (strict comparison)
null == undefined  // true
null === undefined // false
```

---

### Destructuring

```javascript
// Array destructuring
const [a, b, c] = [1, 2, 3];
const [first, ...rest] = [1, 2, 3, 4];  // first=1, rest=[2,3,4]
const [x = 10] = [];  // x=10 (default value)

// Object destructuring
const { name, age } = { name: 'Alice', age: 30 };
const { name: userName } = { name: 'Alice' };  // Rename
const { name, ...others } = { name: 'Alice', age: 30, city: 'NYC' };
const { name = 'Unknown' } = {};  // Default value

// Nested destructuring
const { address: { city } } = { address: { city: 'NYC' } };

// Function parameters
function greet({ name, age }) {
    return `Hello ${name}, age ${age}`;
}
greet({ name: 'Alice', age: 30 });
```

---

### Spread & Rest Operators

```javascript
// Spread (expand)
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4];  // [1, 2, 3, 4]

const obj1 = { a: 1 };
const obj2 = { ...obj1, b: 2 };  // { a: 1, b: 2 }

// Rest (collect)
function sum(...numbers) {
    return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4);  // 10

const [first, ...rest] = [1, 2, 3];  // first=1, rest=[2,3]
```

---

### Template Literals

```javascript
const name = 'Alice';
const greeting = `Hello, ${name}!`;  // "Hello, Alice!"

// Multi-line
const html = `
    <div>
        <h1>${name}</h1>
        <p>Welcome!</p>
    </div>
`;

// Tagged templates
function highlight(strings, ...values) {
    return strings.reduce((result, str, i) => 
        `${result}${str}<mark>${values[i] || ''}</mark>`, '');
}
const highlighted = highlight`Hello ${name}, you are ${age} years old`;
```

---

### Arrow Functions

```javascript
// Traditional function
function add(a, b) {
    return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// Multiple statements
const process = (x) => {
    const doubled = x * 2;
    return doubled + 1;
};

// Returning objects
const getUser = (name) => ({ name, id: Math.random() });

// this binding difference
const obj = {
    name: 'Alice',
    
    // Traditional: this = obj
    greet() {
        console.log(this.name);  // 'Alice'
    },
    
    // Arrow: this = outer scope (NOT obj)
    greetArrow: () => {
        console.log(this.name);  // undefined (window.name)
    }
};
```

---

### Array Methods

```javascript
const numbers = [1, 2, 3, 4, 5];

// map: transform each element
const doubled = numbers.map(n => n * 2);  // [2, 4, 6, 8, 10]

// filter: keep elements that pass test
const evens = numbers.filter(n => n % 2 === 0);  // [2, 4]

// reduce: accumulate to single value
const sum = numbers.reduce((acc, n) => acc + n, 0);  // 15

// find: first element that passes test
const found = numbers.find(n => n > 3);  // 4

// findIndex: index of first element that passes test
const index = numbers.findIndex(n => n > 3);  // 3

// some: at least one element passes test
const hasEven = numbers.some(n => n % 2 === 0);  // true

// every: all elements pass test
const allPositive = numbers.every(n => n > 0);  // true

// includes: array contains value
const hasThree = numbers.includes(3);  // true

// flat: flatten nested arrays
const nested = [[1, 2], [3, [4, 5]]];
const flat = nested.flat(Infinity);  // [1, 2, 3, 4, 5]

// Chaining
const result = numbers
    .filter(n => n > 2)
    .map(n => n * 10)
    .reduce((acc, n) => acc + n, 0);  // 120
```

---

### Object Methods

```javascript
const user = { name: 'Alice', age: 30, city: 'NYC' };

// Keys, values, entries
Object.keys(user);    // ['name', 'age', 'city']
Object.values(user);  // ['Alice', 30, 'NYC']
Object.entries(user); // [['name', 'Alice'], ['age', 30], ['city', 'NYC']]

// Spread for cloning (shallow)
const clone = { ...user };

// Object.assign
const merged = Object.assign({}, user, { age: 31 });

// Optional chaining
const street = user?.address?.street;  // undefined (no error)

// Nullish coalescing
const name = user.name ?? 'Unknown';  // 'Alice'
const nickname = user.nickname ?? 'Unknown';  // 'Unknown'
```

---

### Interview Questions

**Q: What's the difference between `var`, `let`, and `const`?**

A: "`var` is function-scoped, hoisted, can be redeclared. `let` is block-scoped, not hoisted (temporal dead zone), can't be redeclared. `const` is block-scoped, must be initialized, can't be reassigned (but objects can be mutated)."

**Q: What's the difference between `==` and `===`?**

A: "`==` performs type coercion before comparison (5 == '5' is true). `===` compares without coercion (5 === '5' is false). Always use `===` to avoid unexpected behavior."

**Q: What's destructuring?**

A: "Extracting values from arrays or objects into variables. Array: const [a, b] = [1, 2]. Object: const { name, age } = obj. Supports defaults, renaming, and nesting."

**Q: What's the spread operator?**

A: "Expands an iterable (array, string, object) into individual elements. Used for copying, merging, and passing arguments. const copy = [...arr], const merged = { ...obj1, ...obj2 }."

---

*Next: [06 — JavaScript Advanced](06-JS-Advanced.md)*
