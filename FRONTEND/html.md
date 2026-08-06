# 01 — HTML Deep Dive

## Semantic HTML, Accessibility, Forms

---

### What is HTML?

**HTML (HyperText Markup Language)** is the standard markup language for creating web pages. It defines the structure and content of a web page.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Page</title>
</head>
<body>
    <header>
        <nav>
            <a href="/">Home</a>
        </nav>
    </header>
    <main>
        <h1>Hello World</h1>
        <p>This is a paragraph.</p>
    </main>
    <footer>
        <p>&copy; 2024</p>
    </footer>
</body>
</html>
```

---

### Semantic HTML

Using HTML elements that convey meaning, not just appearance.

```html
<!-- BAD: Non-semantic -->
<div class="header">
    <div class="nav">
        <div class="nav-item">Home</div>
    </div>
</div>
<div class="main">
    <div class="article">
        <div class="title">Article Title</div>
    </div>
</div>

<!-- GOOD: Semantic -->
<header>
    <nav>
        <a href="/">Home</a>
    </nav>
</header>
<main>
    <article>
        <h1>Article Title</h1>
    </article>
</main>
```

#### Common Semantic Elements

| Element | Purpose |
|---------|---------|
| `<header>` | Page or section header |
| `<nav>` | Navigation links |
| `<main>` | Main content (one per page) |
| `<article>` | Independent content |
| `<section>` | Thematic grouping |
| `<aside>` | Sidebar, tangential content |
| `<footer>` | Page or section footer |
| `<figure>` | Illustration, diagram |
| `<figcaption>` | Caption for figure |
| `<details>` | Expandable disclosure |
| `<summary>` | Summary for details |
| `<mark>` | Highlighted text |
| `<time>` | Date/time |
| `<address>` | Contact information |

#### Why Semantic HTML Matters

| Benefit | Explanation |
|---------|-------------|
| **Accessibility** | Screen readers understand structure |
| **SEO** | Search engines understand content |
| **Maintainability** | Code is self-documenting |
| **Consistency** | Browser default styles are meaningful |

---

### HTML Forms

```html
<form action="/submit" method="POST">
    <!-- Text input -->
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" required minlength="2" maxlength="50">
    
    <!-- Email -->
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    
    <!-- Password -->
    <label for="password">Password:</label>
    <input type="password" id="password" name="password" required minlength="8">
    
    <!-- Number -->
    <label for="age">Age:</label>
    <input type="number" id="age" name="age" min="0" max="150">
    
    <!-- Date -->
    <label for="dob">Date of Birth:</label>
    <input type="date" id="dob" name="dob">
    
    <!-- Radio buttons -->
    <fieldset>
        <legend>Gender:</legend>
        <input type="radio" id="male" name="gender" value="male">
        <label for="male">Male</label>
        <input type="radio" id="female" name="gender" value="female">
        <label for="female">Female</label>
    </fieldset>
    
    <!-- Checkboxes -->
    <input type="checkbox" id="terms" name="terms" required>
    <label for="terms">I agree to terms</label>
    
    <!-- Select dropdown -->
    <label for="country">Country:</label>
    <select id="country" name="country">
        <option value="">Select...</option>
        <option value="us">United States</option>
        <option value="uk">United Kingdom</option>
    </select>
    
    <!-- Textarea -->
    <label for="bio">Bio:</label>
    <textarea id="bio" name="bio" rows="4" maxlength="500"></textarea>
    
    <!-- File upload -->
    <label for="avatar">Avatar:</label>
    <input type="file" id="avatar" name="avatar" accept="image/*">
    
    <!-- Submit -->
    <button type="submit">Submit</button>
</form>
```

---

### Accessibility (a11y)

```html
<!-- Use ARIA labels when native HTML isn't enough -->
<button aria-label="Close dialog" onclick="close()">X</button>

<!-- Use alt text for images -->
<img src="dog.jpg" alt="A golden retriever playing in the park">

<!-- Use role for custom widgets -->
<div role="alert">Error: Invalid email</div>

<!-- Use aria-live for dynamic content -->
<div aria-live="polite" id="status"></div>

<!-- Use tabindex for keyboard navigation -->
<div tabindex="0" role="button" onclick="handleClick()">Click me</div>

<!-- Use proper heading hierarchy -->
<h1>Main Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>
```

#### Accessibility Checklist

| Item | Description |
|------|-------------|
| **Alt text** | All images have descriptive alt text |
| **Labels** | All form inputs have associated labels |
| **Headings** | Proper heading hierarchy (h1 → h2 → h3) |
| **Color contrast** | Sufficient contrast ratio (4.5:1 minimum) |
| **Keyboard navigation** | All interactive elements are keyboard accessible |
| **Focus indicators** | Visible focus states for keyboard users |
| **ARIA labels** | Used when semantic HTML isn't sufficient |
| **Language** | `<html lang="en">` attribute set |

---

### Meta Tags

```html
<head>
    <!-- Character encoding -->
    <meta charset="UTF-8">
    
    <!-- Viewport for responsive design -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- SEO -->
    <meta name="description" content="Page description for search engines">
    <meta name="keywords" content="keyword1, keyword2">
    <meta name="author" content="Author Name">
    
    <!-- Open Graph (Facebook, LinkedIn) -->
    <meta property="og:title" content="Page Title">
    <meta property="og:description" content="Page description">
    <meta property="og:image" content="https://example.com/image.jpg">
    <meta property="og:url" content="https://example.com">
    
    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:title" content="Page Title">
    
    <!-- Favicon -->
    <link rel="icon" href="/favicon.ico">
    
    <title>Page Title</title>
</head>
```

---

### Interview Questions

**Q: What is semantic HTML and why is it important?**

A: "Semantic HTML uses elements that convey meaning—header, nav, main, article, footer—instead of generic divs. It's important for accessibility (screen readers understand structure), SEO (search engines understand content), and maintainability (code is self-documenting)."

**Q: What's the difference between `<div>` and `<section>`?**

A: "A div is a generic container with no semantic meaning—used for styling/layout. A section represents a thematic grouping of content with a heading. Use section when the content has a clear topic; use div for purely structural/styling purposes."

**Q: How do you make a website accessible?**

A: "Use semantic HTML, provide alt text for images, ensure keyboard navigation, maintain proper heading hierarchy, use ARIA labels when needed, ensure sufficient color contrast, and test with screen readers."

**Q: What's the difference between `GET` and `POST` form methods?**

A: "GET appends form data to URL as query parameters—visible in URL, bookmarkable, limited size, used for searches. POST sends data in request body—not visible, no size limit, used for form submissions that modify data."

---

*Next: [02 — CSS Deep Dive](02-CSS.md)*
