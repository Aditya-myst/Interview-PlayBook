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

*Next: [03 — Authentication Flow](03-Auth-Flow.md)*
