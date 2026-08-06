# 07 — Async JavaScript

## Promises, Async/Await, Callbacks

---

### Callbacks

A function passed as an argument to another function.

```javascript
// Callback pattern
function fetchData(callback) {
    setTimeout(() => {
        callback(null, { name: 'Alice' });
    }, 1000);
}

fetchData((err, data) => {
    if (err) {
        console.error(err);
        return;
    }
    console.log(data);
});

// Callback hell (pyramid of doom)
getUser(userId, (err, user) => {
    getOrders(user.id, (err, orders) => {
        getOrderDetails(orders[0].id, (err, details) => {
            console.log(details);
        });
    });
});
```

---

### Promises

An object representing the eventual completion or failure of an async operation.

```javascript
// Creating a promise
const promise = new Promise((resolve, reject) => {
    const success = true;
    
    if (success) {
        resolve('Data loaded');
    } else {
        reject(new Error('Failed'));
    }
});

// Consuming a promise
promise
    .then(data => console.log(data))    // 'Data loaded'
    .catch(err => console.error(err))
    .finally(() => console.log('Done'));

// Chaining promises
fetchUser(userId)
    .then(user => fetchOrders(user.id))
    .then(orders => fetchDetails(orders[0].id))
    .then(details => console.log(details))
    .catch(err => console.error(err));
```

#### Promise Methods

```javascript
// Promise.all - wait for all (fails if any fail)
const results = await Promise.all([
    fetch('/api/users'),
    fetch('/api/posts'),
    fetch('/api/comments')
]);

// Promise.allSettled - wait for all (never fails)
const results = await Promise.allSettled([
    fetch('/api/users'),
    fetch('/api/posts'),
    fetch('/api/invalid')  // Even if this fails
]);
// [{status: 'fulfilled', value: ...}, {status: 'rejected', reason: ...}]

// Promise.race - first to complete (success or failure)
const fastest = await Promise.race([
    fetch('/api/server1'),
    fetch('/api/server2')
]);

// Promise.any - first to succeed (ignores failures)
const result = await Promise.any([
    fetch('/api/server1'),
    fetch('/api/server2')
]);
```

---

### Async/Await

Syntactic sugar over Promises. Makes async code look synchronous.

```javascript
// Basic async function
async function fetchUser(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        const user = await response.json();
        return user;
    } catch (error) {
        console.error('Failed to fetch user:', error);
        throw error;
    }
}

// Using async function
const user = await fetchUser(123);

// Sequential vs Parallel
// Sequential (slow - waits for each)
const users = await fetchUsers();
const posts = await fetchPosts();

// Parallel (fast - runs simultaneously)
const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts()
]);

// Error handling
async function getData() {
    try {
        const data = await riskyOperation();
        return data;
    } catch (error) {
        // Handle error
        return defaultValue;
    } finally {
        // Cleanup
    }
}
```

---

### Interview Questions

**Q: What's the difference between callbacks and Promises?**

A: "Callbacks: functions passed as arguments, can lead to callback hell. Promises: objects representing future values, chainable with .then(), better error handling. Async/await is syntactic sugar over Promises."

**Q: What's the event loop and how does async work?**

A: "JavaScript is single-threaded. Async operations (setTimeout, fetch) are handled by browser APIs. When complete, callbacks go to task queues. Microtasks (Promises) have priority over macrotasks (setTimeout)."

**Q: What's the difference between Promise.all and Promise.allSettled?**

A: "Promise.all: waits for all promises, rejects if ANY fail. Promise.allSettled: waits for all promises, never rejects, returns status of each. Use all when you need all to succeed; allSettled when you want all results regardless."

**Q: How do you handle errors in async/await?**

A: "Use try/catch blocks. Wrap await calls in try, catch errors in catch block. For parallel operations, use Promise.allSettled to handle partial failures. Always handle errors—unhandled promise rejections crash Node.js."

---

*Next: [08 — TypeScript](08-TypeScript.md)*
