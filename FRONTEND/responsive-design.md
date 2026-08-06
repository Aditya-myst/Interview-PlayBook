# 04 — Responsive Design

## Media Queries, Mobile-First, Breakpoints

---

### What is Responsive Design?

Responsive design ensures web pages look good on all devices—phones, tablets, desktops.

---

### Mobile-First Approach

Design for mobile first, then add styles for larger screens.

```css
/* Mobile-first: base styles are for mobile */
.container {
    padding: 16px;
    font-size: 14px;
}

/* Tablet and up */
@media (min-width: 768px) {
    .container {
        padding: 24px;
        font-size: 16px;
    }
}

/* Desktop and up */
@media (min-width: 1024px) {
    .container {
        padding: 32px;
        max-width: 1200px;
        margin: 0 auto;
    }
}
```

---

### Media Queries

```css
/* Breakpoints */
@media (min-width: 480px) { }   /* Small phones */
@media (min-width: 768px) { }   /* Tablets */
@media (min-width: 1024px) { }  /* Small desktops */
@media (min-width: 1200px) { }  /* Large desktops */

/* Orientation */
@media (orientation: portrait) { }
@media (orientation: landscape) { }

/* Dark mode */
@media (prefers-color-scheme: dark) {
    body { background: #1a1a1a; color: white; }
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
    * { animation: none !important; transition: none !important; }
}

/* Print */
@media print {
    nav, footer { display: none; }
}
```

---

### Responsive Images

```html
<!-- Srcset for different resolutions -->
<img src="small.jpg" 
     srcset="small.jpg 480w, medium.jpg 768w, large.jpg 1200w"
     sizes="(max-width: 480px) 100vw, (max-width: 768px) 50vw, 33vw"
     alt="Responsive image">

<!-- Picture element for art direction -->
<picture>
    <source media="(min-width: 1024px)" srcset="desktop.jpg">
    <source media="(min-width: 768px)" srcset="tablet.jpg">
    <img src="mobile.jpg" alt="Responsive">
</picture>
```

```css
/* Responsive images */
img {
    max-width: 100%;
    height: auto;
}
```

---

### Responsive Typography

```css
/* Using clamp() for fluid typography */
h1 {
    font-size: clamp(1.5rem, 4vw, 3rem);  /* Min, preferred, max */
}

p {
    font-size: clamp(0.875rem, 2vw, 1.125rem);
}

/* Using rem units */
body {
    font-size: 16px;  /* Base */
}

h1 { font-size: 2rem; }    /* 32px */
h2 { font-size: 1.5rem; }  /* 24px */
p { font-size: 1rem; }     /* 16px */
```

---

### Responsive Layouts

```css
/* Responsive grid */
.grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 16px;
}

/* Responsive flexbox */
.flex-container {
    display: flex;
    flex-wrap: wrap;
    gap: 16px;
}

.flex-item {
    flex: 1 1 300px;  /* Grow, shrink, min-width 300px */
}

/* Hide/show elements */
.mobile-only { display: block; }
.desktop-only { display: none; }

@media (min-width: 768px) {
    .mobile-only { display: none; }
    .desktop-only { display: block; }
}
```

---

### Interview Questions

**Q: What's mobile-first design?**

A: "Design for mobile devices first, then add styles for larger screens using min-width media queries. Benefits: forces you to prioritize content, better performance on mobile (loads less CSS), progressive enhancement."

**Q: What's the difference between `min-width` and `max-width` media queries?**

A: "min-width: styles apply when viewport is AT LEAST that width (mobile-first, additive). max-width: styles apply when viewport is AT MOST that width (desktop-first, subtractive). Mobile-first uses min-width."

**Q: How do you make images responsive?**

A: "Set max-width: 100% and height: auto. Use srcset for different resolutions, sizes for viewport-based selection, and picture element for art direction (different crops for different screens)."

---

*Next: [05 — JavaScript Fundamentals](05-JS-Fundamentals.md)*
