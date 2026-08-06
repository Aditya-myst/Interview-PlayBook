# 10 — React Advanced

## Performance, Patterns, Custom Hooks

---

### React.memo

Prevents re-rendering if props haven't changed.

```jsx
const UserCard = React.memo(function UserCard({ name, email }) {
    console.log('UserCard rendered');
    return (
        <div>
            <h3>{name}</h3>
            <p>{email}</p>
        </div>
    );
});

// Only re-renders when name or email changes
```

---

### Custom Hooks

```jsx
// useLocalStorage
function useLocalStorage(key, initialValue) {
    const [value, setValue] = useState(() => {
        const saved = localStorage.getItem(key);
        return saved ? JSON.parse(saved) : initialValue;
    });
    
    useEffect(() => {
        localStorage.setItem(key, JSON.stringify(value));
    }, [key, value]);
    
    return [value, setValue];
}

// useFetch
function useFetch(url) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        let cancelled = false;
        
        async function fetchData() {
            try {
                setLoading(true);
                const response = await fetch(url);
                const json = await response.json();
                if (!cancelled) setData(json);
            } catch (err) {
                if (!cancelled) setError(err);
            } finally {
                if (!cancelled) setLoading(false);
            }
        }
        
        fetchData();
        return () => { cancelled = true; };
    }, [url]);
    
    return { data, loading, error };
}

// useDebounce
function useDebounce(value, delay) {
    const [debouncedValue, setDebouncedValue] = useState(value);
    
    useEffect(() => {
        const timer = setTimeout(() => setDebouncedValue(value), delay);
        return () => clearTimeout(timer);
    }, [value, delay]);
    
    return debouncedValue;
}

// Usage
function SearchInput() {
    const [query, setQuery] = useState('');
    const debouncedQuery = useDebounce(query, 300);
    const { data, loading } = useFetch(`/api/search?q=${debouncedQuery}`);
    
    return (
        <div>
            <input value={query} onChange={e => setQuery(e.target.value)} />
            {loading ? <p>Loading...</p> : <Results data={data} />}
        </div>
    );
}
```

---

### Error Boundaries

```jsx
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
    }
    
    static getDerivedStateFromError(error) {
        return { hasError: true, error };
    }
    
    componentDidCatch(error, errorInfo) {
        console.error('Error caught:', error, errorInfo);
    }
    
    render() {
        if (this.state.hasError) {
            return (
                <div>
                    <h2>Something went wrong</h2>
                    <p>{this.state.error?.message}</p>
                    <button onClick={() => this.setState({ hasError: false })}>
                        Try again
                    </button>
                </div>
            );
        }
        return this.props.children;
    }
}

// Usage
<ErrorBoundary>
    <App />
</ErrorBoundary>
```

---

### Performance Optimization

```jsx
// 1. Code splitting with lazy loading
const Dashboard = React.lazy(() => import('./Dashboard'));

function App() {
    return (
        <Suspense fallback={<Loading />}>
            <Dashboard />
        </Suspense>
    );
}

// 2. Virtualized lists for large data
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
    return (
        <FixedSizeList height={400} itemCount={items.length} itemSize={50}>
            {({ index, style }) => (
                <div style={style}>{items[index].name}</div>
            )}
        </FixedSizeList>
    );
}

// 3. Avoid inline functions in JSX
// BAD
<button onClick={() => handleClick(id)}>Click</button>

// GOOD
const handleItemClick = useCallback((id) => handleClick(id), []);
<button onClick={handleItemClick}>Click</button>

// 4. Debounce expensive operations
const debouncedSearch = useDebounce(searchTerm, 300);
```

---

### State Management Patterns

```jsx
// useReducer for complex state
function todoReducer(state, action) {
    switch (action.type) {
        case 'ADD':
            return [...state, { id: Date.now(), text: action.text, done: false }];
        case 'TOGGLE':
            return state.map(todo =>
                todo.id === action.id ? { ...todo, done: !todo.done } : todo
            );
        case 'DELETE':
            return state.filter(todo => todo.id !== action.id);
        default:
            return state;
    }
}

function TodoApp() {
    const [todos, dispatch] = useReducer(todoReducer, []);
    
    return (
        <div>
            <button onClick={() => dispatch({ type: 'ADD', text: 'New todo' })}>
                Add
            </button>
            {todos.map(todo => (
                <div key={todo.id}>
                    <span style={{ textDecoration: todo.done ? 'line-through' : 'none' }}>
                        {todo.text}
                    </span>
                    <button onClick={() => dispatch({ type: 'TOGGLE', id: todo.id })}>
                        Toggle
                    </button>
                </div>
            ))}
        </div>
    );
}
```

---

### Interview Questions

**Q: What's React.memo and when would you use it?**

A: "A higher-order component that prevents re-rendering if props haven't changed (shallow comparison). Use for expensive components that receive the same props frequently. Don't use for components that almost always receive different props."

**Q: What are custom hooks?**

A: "Functions that start with 'use' and can call other hooks. Extract reusable logic from components. Examples: useFetch, useLocalStorage, useDebounce. They share logic, not state—each component gets its own state."

**Q: How do you handle errors in React?**

A: "Error boundaries (class components) catch rendering errors. For async errors, use try/catch in useEffect or event handlers. Show fallback UI, provide retry mechanism, log errors for debugging."

---

*Next: [11 — Next.js](11-NextJS.md)*
