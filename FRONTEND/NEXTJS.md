# 11 — Next.js

## SSR, SSG, Routing, API Routes

---

### What is Next.js?

A React framework with server-side rendering, static generation, file-based routing, and API routes built in.

---

### Rendering Strategies

| Strategy | When HTML is Generated | Use Case |
|----------|----------------------|----------|
| **CSR** (Client-Side) | In browser | SPAs, dashboards |
| **SSR** (Server-Side) | On each request | Dynamic content, personalized pages |
| **SSG** (Static Site) | At build time | Blog posts, marketing pages |
| **ISR** (Incremental) | At build + revalidation | Content that changes occasionally |

```jsx
// SSR - runs on every request
export async function getServerSideProps(context) {
    const data = await fetch('https://api.example.com/data');
    const json = await data.json();
    
    return {
        props: { data: json }
    };
}

// SSG - runs at build time
export async function getStaticProps() {
    const data = await fetch('https://api.example.com/data');
    const json = await data.json();
    
    return {
        props: { data: json },
        revalidate: 60  // ISR: revalidate every 60 seconds
    };
}

// Dynamic SSG
export async function getStaticPaths() {
    return {
        paths: [
            { params: { id: '1' } },
            { params: { id: '2' } }
        ],
        fallback: false  // or true, 'blocking'
    };
}
```

---

### File-Based Routing

```
app/
├── page.js              → /
├── about/
│   └── page.js          → /about
├── blog/
│   ├── page.js          → /blog
│   └── [slug]/
│       └── page.js      → /blog/:slug
├── api/
│   └── users/
│       └── route.js     → /api/users
└── layout.js            → Root layout
```

```jsx
// app/page.js (Home)
export default function Home() {
    return <h1>Home Page</h1>;
}

// app/blog/[slug]/page.js (Dynamic route)
export default function BlogPost({ params }) {
    return <h1>Post: {params.slug}</h1>;
}

// app/layout.js (Shared layout)
export default function RootLayout({ children }) {
    return (
        <html>
            <body>
                <nav>...</nav>
                {children}
                <footer>...</footer>
            </body>
        </html>
    );
}
```

---

### API Routes

```jsx
// app/api/users/route.js
import { NextResponse } from 'next/server';

export async function GET(request) {
    const users = await db.getUsers();
    return NextResponse.json(users);
}

export async function POST(request) {
    const body = await request.json();
    const user = await db.createUser(body);
    return NextResponse.json(user, { status: 201 });
}

// app/api/users/[id]/route.js
export async function GET(request, { params }) {
    const user = await db.getUser(params.id);
    if (!user) {
        return NextResponse.json({ error: 'Not found' }, { status: 404 });
    }
    return NextResponse.json(user);
}
```

---

### Server Components vs Client Components

```jsx
// Server Component (default in App Router)
// Runs on server, can access DB directly, no client JS
export default async function UserProfile({ userId }) {
    const user = await db.getUser(userId);  // Direct DB access!
    
    return (
        <div>
            <h1>{user.name}</h1>
            <p>{user.email}</p>
        </div>
    );
}

// Client Component
'use client';
import { useState } from 'react';

export default function Counter() {
    const [count, setCount] = useState(0);
    
    return (
        <button onClick={() => setCount(count + 1)}>
            Count: {count}
        </button>
    );
}
```

---

### Metadata & SEO

```jsx
// app/page.js
export const metadata = {
    title: 'My Page',
    description: 'Page description',
    openGraph: {
        title: 'My Page',
        description: 'Page description',
        images: ['/og-image.jpg'],
    },
};

// Dynamic metadata
export async function generateMetadata({ params }) {
    const post = await getPost(params.slug);
    return {
        title: post.title,
        description: post.excerpt,
    };
}
```

---

### Interview Questions

**Q: What's the difference between SSR and SSG?**

A: "SSR: HTML generated on each request—good for dynamic/personalized content. SSG: HTML generated at build time—good for static content, faster. ISR: SSG with periodic revalidation."

**Q: What are Server Components?**

A: "Components that run on the server—can access databases directly, reduce client-side JavaScript, improve performance. They can't use hooks or browser APIs. Use 'use client' directive for client components."

**Q: When would you use Next.js over Create React App?**

A: "When you need SSR/SSG for SEO, file-based routing, API routes, or image optimization. CRA is simpler but only supports client-side rendering. Next.js is production-ready with better performance."

---

*Next: [12 — Frontend Interview Q&A](12-Interview-QA.md)*
