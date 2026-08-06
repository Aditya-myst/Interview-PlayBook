# 09 — React Fundamentals

## Components, Hooks, JSX, State

---

### What is React?

A JavaScript library for building user interfaces. Components, virtual DOM, declarative.

---

### JSX

```jsx
// JSX is HTML-like syntax in JavaScript
const element = <h1>Hello, {name}!</h1>;

// Conditional rendering
const greeting = isLoggedIn ? <h1>Welcome back!</h1> : <h1>Please log in</h1>;

// Lists
const items = ['Apple', 'Banana', 'Cherry'];
const list = (
    <ul>
        {items.map((item, index) => (
            <li key={index}>{item}</li>
        ))}
    </ul>
);

// Attributes
const img = <img src="photo.jpg" alt="A photo" className="rounded" />;
const button = <button onClick={handleClick}>Click me</button>;
```

---

### Components

```jsx
// Function Component (modern)
function Welcome({ name }) {
    return <h1>Hello, {name}!</h1>;
}

// With TypeScript
interface WelcomeProps {
    name: string;
    age?: number;
}

function Welcome({ name, age }: WelcomeProps) {
    return <h1>Hello, {name}! {age && `Age: ${age}`}</h1>;
}

// Children
function Card({ title, children }) {
    return (
        <div className="card">
            <h2>{title}</h2>
            <div>{children}</div>
        </div>
    );
}

// Usage
<Card title="User Profile">
    <p>Name: Alice</p>
    <p>Age: 30</p>
</Card>
```

---

### useState Hook

```jsx
import { useState } from 'react';

function Counter() {
    const [count, setCount] = useState(0);
    
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>Increment</button>
            <button onClick={() => setCount(prev => prev + 1)}>Increment (functional)</button>
        </div>
    );
}

// Complex state
function Form() {
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        age: 0
    });
    
    const handleChange = (e) => {
        setFormData(prev => ({
            ...prev,
            [e.target.name]: e.target.value
        }));
    };
    
    return (
        <form>
            <input name="name" value={formData.name} onChange={handleChange} />
            <input name="email" value={formData.email} onChange={handleChange} />
        </form>
    );
}
```

---

### useEffect Hook

```jsx
import { useEffect, useState } from 'react';

function UserProfile({ userId }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    
    // Runs after render, when userId changes
    useEffect(() => {
        let cancelled = false;
        
        async function fetchUser() {
            setLoading(true);
            try {
                const response = await fetch(`/api/users/${userId}`);
                const data = await response.json();
                if (!cancelled) {
                    setUser(data);
                }
            } finally {
                if (!cancelled) {
                    setLoading(false);
                }
            }
        }
        
        fetchUser();
        
        // Cleanup function
        return () => {
            cancelled = true;
        };
    }, [userId]);  // Dependency array
    
    if (loading) return <div>Loading...</div>;
    if (!user) return <div>User not found</div>;
    
    return <div>{user.name}</div>;
}
```

#### useEffect Dependency Array

```jsx
// No dependency array: runs after every render
useEffect(() => {
    console.log('Runs after every render');
});

// Empty dependency array: runs once on mount
useEffect(() => {
    console.log('Runs once on mount');
}, []);

// With dependencies: runs when dependencies change
useEffect(() => {
    console.log('Runs when count changes');
}, [count]);
```

---

### useRef Hook

```jsx
import { useRef } from 'react';

function TextInput() {
    const inputRef = useRef(null);
    
    const focusInput = () => {
        inputRef.current.focus();
    };
    
    return (
        <div>
            <input ref={inputRef} type="text" />
            <button onClick={focusInput}>Focus Input</button>
        </div>
    );
}

// Store mutable values (doesn't cause re-render)
function Timer() {
    const [count, setCount] = useState(0);
    const intervalRef = useRef(null);
    
    useEffect(() => {
        intervalRef.current = setInterval(() => {
            setCount(prev => prev + 1);
        }, 1000);
        
        return () => clearInterval(intervalRef.current);
    }, []);
    
    return <div>Count: {count}</div>;
}
```

---

### useMemo & useCallback

```jsx
import { useMemo, useCallback } from 'react';

function ExpensiveComponent({ items, onItemClick }) {
    // Memoize expensive computation
    const sortedItems = useMemo(() => {
        return [...items].sort((a, b) => a.name.localeCompare(b.name));
    }, [items]);  // Only recompute when items change
    
    // Memoize callback (prevent child re-renders)
    const handleClick = useCallback((id) => {
        onItemClick(id);
    }, [onItemClick]);
    
    return (
        <ul>
            {sortedItems.map(item => (
                <li key={item.id} onClick={() => handleClick(item.id)}>
                    {item.name}
                </li>
            ))}
        </ul>
    );
}
```

---

### Context API

```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const ThemeContext = createContext();

// Provider
function ThemeProvider({ children }) {
    const [theme, setTheme] = useState('light');
    
    const toggleTheme = () => {
        setTheme(prev => prev === 'light' ? 'dark' : 'light');
    };
    
    return (
        <ThemeContext.Provider value={{ theme, toggleTheme }}>
            {children}
        </ThemeContext.Provider>
    );
}

// Consumer
function ThemedButton() {
    const { theme, toggleTheme } = useContext(ThemeContext);
    
    return (
        <button 
            onClick={toggleTheme}
            style={{ background: theme === 'light' ? '#fff' : '#333' }}
        >
            Toggle Theme ({theme})
        </button>
    );
}

// Usage
function App() {
    return (
        <ThemeProvider>
            <ThemedButton />
        </ThemeProvider>
    );
}
```

---

### Event Handling

```jsx
function Form() {
    const [name, setName] = useState('');
    
    // Inline handler
    const handleClick = () => {
        console.log('Clicked!');
    };
    
    // With parameters
    const handleDelete = (id) => {
        console.log(`Delete item ${id}`);
    };
    
    // Form submission
    const handleSubmit = (e) => {
        e.preventDefault();
        console.log('Submitted:', name);
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <input 
                value={name}
                onChange={(e) => setName(e.target.value)}
            />
            <button type="submit">Submit</button>
            <button onClick={handleClick}>Click</button>
            <button onClick={() => handleDelete(123)}>Delete</button>
        </form>
    );
}
```

---

### Interview Questions

**Q: What is React?**

A: "A JavaScript library for building UIs. Key concepts: components (reusable UI pieces), virtual DOM (efficient updates), declarative (describe what, not how), unidirectional data flow."

**Q: What's the virtual DOM?**

A: "An in-memory representation of the real DOM. When state changes, React creates a new virtual DOM, compares it with the previous (diffing), and updates only the changed parts in the real DOM (reconciliation). This is faster than direct DOM manipulation."

**Q: What's the difference between `useMemo` and `useCallback`?**

A: "useMemo memoizes a VALUE (result of computation). useCallback memoizes a FUNCTION (reference). useMemo(() => expensiveCompute(a), [a]) returns cached result. useCallback((id) => doSomething(id), []) returns cached function reference."

**Q: When does a component re-render?**

A: "When state changes, props change, or parent re-renders. Prevent unnecessary re-renders with React.memo, useMemo, useCallback. Context changes also trigger re-renders for all consumers."

---

*Next: [10 — React Advanced](10-React-Advanced.md)*
