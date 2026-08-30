# 02 — API Integration

## Connecting Frontend to Backend

---

### Fetch API (Native)

```javascript
// GET request
async function getUsers() {
    const response = await fetch('/api/users');
    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
}

// POST request
async function createUser(userData) {
    const response = await fetch('/api/users', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(userData),
    });
    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json();
}

// With authentication
async function getProtectedData() {
    const token = localStorage.getItem('accessToken');
    const response = await fetch('/api/protected', {
        headers: {
            'Authorization': `Bearer ${token}`,
        },
    });
    return response.json();
}
```

---

### Axios

```javascript
import axios from 'axios';

// Create instance with defaults
const api = axios.create({
    baseURL: 'https://api.example.com',
    timeout: 10000,
    headers: {
        'Content-Type': 'application/json',
    },
});

// Request interceptor (add auth token)
api.interceptors.request.use(
    (config) => {
        const token = localStorage.getItem('accessToken');
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
    },
    (error) => Promise.reject(error)
);

// Response interceptor (handle token refresh)
api.interceptors.response.use(
    (response) => response,
    async (error) => {
        if (error.response?.status === 401) {
            // Try to refresh token
            const refreshed = await refreshToken();
            if (refreshed) {
                // Retry original request
                return api(error.config);
            }
        }
        return Promise.reject(error);
    }
);

// Usage
const users = await api.get('/users');
const newUser = await api.post('/users', { name: 'Alice' });
await api.put('/users/1', { name: 'Alice Smith' });
await api.delete('/users/1');
```

---

### React Query (TanStack Query)

```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Fetch data
function UsersList() {
    const { data: users, isLoading, error } = useQuery({
        queryKey: ['users'],
        queryFn: () => api.get('/users').then(res => res.data),
        staleTime: 5 * 60 * 1000,  // 5 minutes
    });

    if (isLoading) return <div>Loading...</div>;
    if (error) return <div>Error: {error.message}</div>;

    return (
        <ul>
            {users.map(user => (
                <li key={user.id}>{user.name}</li>
            ))}
        </ul>
    );
}

// Create mutation
function CreateUserForm() {
    const queryClient = useQueryClient();
    
    const mutation = useMutation({
        mutationFn: (userData) => api.post('/users', userData),
        onSuccess: () => {
            // Invalidate and refetch
            queryClient.invalidateQueries({ queryKey: ['users'] });
        },
    });

    const handleSubmit = (e) => {
        e.preventDefault();
        mutation.mutate({ name: 'Alice', email: 'alice@example.com' });
    };

    return (
        <form onSubmit={handleSubmit}>
            {mutation.isPending && <p>Creating...</p>}
            {mutation.isError && <p>Error: {mutation.error.message}</p>}
            {mutation.isSuccess && <p>Created!</p>}
            <button type="submit">Create User</button>
        </form>
    );
}
```

---

### Error Handling Patterns

```javascript
// API error class
class ApiError extends Error {
    constructor(message, status, data) {
        super(message);
        this.status = status;
        this.data = data;
    }
}

// API client with error handling
async function apiCall(url, options = {}) {
    try {
        const response = await fetch(url, {
            ...options,
            headers: {
                'Content-Type': 'application/json',
                ...options.headers,
            },
        });

        if (!response.ok) {
            const errorData = await response.json().catch(() => null);
            throw new ApiError(
                errorData?.message || 'Request failed',
                response.status,
                errorData
            );
        }

        return response.json();
    } catch (error) {
        if (error instanceof ApiError) throw error;
        throw new ApiError('Network error', 0, null);
    }
}

// Usage in component
function UserProfile({ userId }) {
    const [user, setUser] = useState(null);
    const [error, setError] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        let cancelled = false;

        async function fetchUser() {
            try {
                setLoading(true);
                setError(null);
                const data = await apiCall(`/api/users/${userId}`);
                if (!cancelled) setUser(data);
            } catch (err) {
                if (!cancelled) setError(err);
            } finally {
                if (!cancelled) setLoading(false);
            }
        }

        fetchUser();
        return () => { cancelled = true; };
    }, [userId]);

    if (loading) return <Spinner />;
    if (error) return <ErrorMessage error={error} />;
    return <UserCard user={user} />;
}
```

---

### Interview Questions

**Q: What's the difference between fetch and axios?**

A: "fetch: native browser API, simpler, doesn't reject on HTTP errors (404/500). axios: library, auto-transforms JSON, interceptors, better error handling, request cancellation. Use axios for complex applications; fetch for simple cases."

**Q: How do you handle API errors in React?**

A: "Try/catch in async functions. Check response.ok for fetch. Use error state to display user-friendly messages. Implement retry logic for transient failures. Show different UI for different error types (404, 500, network)."

**Q: What is React Query and why would you use it?**

A: "Server state management library. Handles caching, background refetching, pagination, optimistic updates. Eliminates boilerplate for data fetching. Provides loading/error states automatically. Better than manual useEffect + useState for API data."

---

### Additional Interview Questions (25+)

**Q4: How do you handle API authentication in frontend?**
A: "Store access token in memory (not localStorage for security). Include in Authorization header. Use refresh token in httpOnly cookie. Implement token refresh logic."

**Q5: What is optimistic updates?**
A: "Update UI immediately before server confirms. If server request fails, rollback. Improves perceived performance. Use with React Query's onMutate."

**Q6: How do you handle API pagination in frontend?**
A: "Infinite scroll with intersection observer. Load more button. Cursor-based or offset-based. React Query's useInfiniteQuery for infinite scroll."

**Q7: What is request cancellation?**
A: "Cancel pending requests when component unmounts or new request starts. AbortController for fetch. CancelToken for axios. Prevents race conditions."

**Q8: How do you handle API caching in frontend?**
A: "React Query for server state caching. SWR as alternative. HTTP caching headers. localStorage for persistent cache. Stale-while-revalidate pattern."

**Q9: What is the difference between server state and client state?**
A: "Server state: data from API, can become stale, needs synchronization. Client state: UI state, form data, local preferences. Use React Query for server state, useState/useReducer for client state."

**Q10: How do you handle API retries?**
A: "Exponential backoff with jitter. Retry on 5xx errors and network failures. Maximum retry count. React Query has built-in retry logic."

**Q11: What is GraphQL and how does it differ from REST?**
A: "GraphQL: single endpoint, client specifies fields, no over/under-fetching. REST: multiple endpoints, fixed response. GraphQL for complex data needs; REST for simple CRUD."

**Q12: How do you handle file uploads in React?**
A: "FormData for multipart uploads. Progress tracking with XMLHttpRequest or axios. Validate file type and size. Show upload progress to user."

**Q13: What is API mocking and when would you use it?**
A: "Simulate API responses for testing. Tools: MSW (Mock Service Worker), json-server. Use for development without backend, testing, demos."

**Q14: How do you handle real-time data in React?**
A: "WebSocket for bidirectional. SSE for server-to-client. React Query for polling. Socket.io for WebSocket abstraction."

**Q15: What is the difference between controlled and uncontrolled components?**
A: "Controlled: React manages form state (value + onChange). Uncontrolled: DOM manages state (ref). Controlled for validation, dynamic forms. Uncontrolled for simple forms."

**Q16: How do you handle form validation in React?**
A: "Libraries: React Hook Form, Formik. Validation: Yup, Zod. Validate on submit and on blur. Show error messages. Server-side validation too."

**Q17: What is React context and when would you use it?**
A: "Share state across components without prop drilling. Use for themes, auth, language. Don't use for frequently changing state (causes re-renders)."

**Q18: How do you handle side effects in React?**
A: "useEffect for side effects. Dependency array controls when it runs. Cleanup function for subscriptions. Don't use for data fetching (use React Query)."

**Q19: What is React.memo and when would you use it?**
A: "Memoize component to prevent re-render if props unchanged. Use for expensive components. Don't use for components that always receive different props."

**Q20: How do you handle global state in React?**
A: "Context API for simple state. Redux for complex state. Zustand for lightweight. React Query for server state. Choose based on complexity."

**Q21: What is code splitting and lazy loading?**
A: "Split code into smaller bundles. Load on demand with React.lazy. Reduces initial load time. Use for routes, heavy components."

**Q22: How do you handle errors in React?**
A: "Error boundaries for rendering errors. try/catch for async. Error state for API calls. Sentry for error tracking."

**Q23: What is React Suspense?**
A: "Show fallback while loading. Use with React.lazy for code splitting. Use with React Query for data fetching. Concurrent features."

**Q24: How do you test React components?**
A: "React Testing Library for unit tests. Jest as test runner. Mock API calls. Test user interactions. Snapshot testing for UI."

**Q25: What is React Server Components?**
A: "Components that run on server. No client JavaScript. Access database directly. Use for static content. Client components for interactivity."

**Q26: How do you handle routing in React?**
A: "React Router for client-side routing. Route parameters, nested routes, protected routes. Next.js for file-based routing."

**Q27: What is React hooks best practices?**
A: "Don't call hooks conditionally. Custom hooks for reusable logic. useEffect cleanup. useMemo/useCallback for optimization."

**Q28: How do you handle authentication in React?**
A: "Auth context for user state. Protected routes. Token storage (memory or httpOnly cookie). Redirect on unauthorized."

**Q29: What is React performance optimization?**
A: "React.memo, useMemo, useCallback. Code splitting. Virtual lists for large data. Avoid unnecessary re-renders."

---

*Next: [03 — Authentication Flow](03-Auth-Flow.md)*
