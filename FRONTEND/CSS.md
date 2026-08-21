# 02 — CSS Deep Dive

## Box Model, Specificity, Positioning

---

### The Box Model

Every HTML element is a rectangular box.

```
┌─────────────────────────────────────┐
│              MARGIN                 │
│  ┌─────────────────────────────┐  │
│  │           BORDER            │  │
│  │  ┌───────────────────────┐  │  │
│  │  │        PADDING        │  │  │
│  │  │  ┌─────────────────┐  │  │  │
│  │  │  │     CONTENT      │  │  │  │
│  │  │  │  (width x height)│  │  │  │
│  │  │  └─────────────────┘  │  │  │
│  │  └───────────────────────┘  │  │
│  └─────────────────────────────┘  │
└─────────────────────────────────────┘
```

```css
/* Box-sizing: border-box (recommended) */
* {
    box-sizing: border-box;
}

/* With border-box, width includes padding and border */
.box {
    width: 200px;       /* Total width = 200px (including padding + border) */
    padding: 20px;
    border: 2px solid black;
    margin: 10px;
}

/* With content-box (default), width is just content */
.box {
    box-sizing: content-box;
    width: 200px;       /* Content width = 200px, total = 200 + 40 + 4 = 244px */
    padding: 20px;
    border: 2px solid black;
}
```

---

### CSS Specificity

When multiple rules target the same element, specificity determines which wins.

```
Specificity hierarchy (highest to lowest):

1. Inline styles          (1,0,0,0)  style="..."
2. ID selectors           (0,1,0,0)  #header
3. Class/attribute/pseudo (0,0,1,0)  .nav, [type="text"], :hover
4. Element/pseudo-element (0,0,0,1)  div, ::before

!important overrides everything (avoid using)
```

```css
/* Specificity examples */
p { color: black; }                    /* 0,0,0,1 */
.content p { color: blue; }           /* 0,0,1,1 - wins over above */
#main .content p { color: green; }    /* 0,1,1,1 - wins over above */
p { color: red !important; }          /* Wins over everything (bad!) */
```

---

### CSS Selectors

```css
/* Element selector */
div { }

/* Class selector */
.container { }

/* ID selector */
#header { }

/* Attribute selector */
input[type="text"] { }
a[href^="https"] { }    /* Starts with */
a[href$=".pdf"] { }     /* Ends with */
a[href*="example"] { }  /* Contains */

/* Combinators */
div p { }       /* Descendant (any depth) */
div > p { }     /* Direct child */
div + p { }     /* Adjacent sibling */
div ~ p { }     /* General sibling */

/* Pseudo-classes */
a:hover { }
a:active { }
a:focus { }
li:first-child { }
li:last-child { }
li:nth-child(2n) { }    /* Even items */
li:nth-child(2n+1) { }  /* Odd items */
p:not(.special) { }

/* Pseudo-elements */
p::first-line { }
p::first-letter { }
p::before { content: "→ "; }
p::after { content: " ←"; }
::selection { background: yellow; }
```

---

### CSS Positioning

```css
/* Static (default) */
.static { position: static; }

/* Relative - relative to its normal position */
.relative { 
    position: relative; 
    top: 10px; 
    left: 20px; 
}

/* Absolute - relative to nearest positioned ancestor */
.absolute { 
    position: absolute; 
    top: 0; 
    right: 0; 
}

/* Fixed - relative to viewport */
.fixed { 
    position: fixed; 
    bottom: 20px; 
    right: 20px; 
}

/* Sticky - hybrid of relative and fixed */
.sticky { 
    position: sticky; 
    top: 0; 
}
```

---

### CSS Units

| Unit | Type | Example |
|------|------|---------|
| `px` | Absolute | `width: 200px` |
| `em` | Relative to parent font | `font-size: 1.5em` |
| `rem` | Relative to root font | `font-size: 1rem` |
| `%` | Relative to parent | `width: 50%` |
| `vw` | Viewport width | `width: 100vw` |
| `vh` | Viewport height | `height: 100vh` |
| `vmin` | Min of vw/vh | `font-size: 5vmin` |
| `vmax` | Max of vw/vh | `font-size: 5vmax` |
| `fr` | Fraction (Grid) | `grid-template-columns: 1fr 2fr` |

---

### CSS Variables (Custom Properties)

```css
:root {
    --primary-color: #3498db;
    --secondary-color: #2ecc71;
    --font-size-base: 16px;
    --spacing-md: 16px;
}

.button {
    background-color: var(--primary-color);
    font-size: var(--font-size-base);
    padding: var(--spacing-md);
}

.button:hover {
    background-color: color-mix(in srgb, var(--primary-color) 80%, black);
}
```

---

### CSS Transitions & Animations

```css
/* Transitions */
.button {
    background: blue;
    transition: background 0.3s ease, transform 0.2s;
}

.button:hover {
    background: darkblue;
    transform: scale(1.05);
}

/* Animations */
@keyframes slideIn {
    from {
        transform: translateX(-100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}

.animated {
    animation: slideIn 0.5s ease-out forwards;
}

/* Animation properties */
.animated {
    animation-name: slideIn;
    animation-duration: 0.5s;
    animation-timing-function: ease-out;
    animation-delay: 0s;
    animation-iteration-count: 1;
    animation-direction: normal;
    animation-fill-mode: forwards;
}
```

---

### Interview Questions

**Q: What's the CSS box model?**

A: "Every element is a rectangular box with content, padding, border, and margin. With box-sizing: border-box, width includes padding and border. With content-box (default), width is just content."

**Q: How does CSS specificity work?**

A: "Specificity determines which CSS rule wins when multiple rules target the same element. Inline styles > IDs > classes/attributes/pseudo-classes > elements/pseudo-elements. !important overrides everything but should be avoided."

**Q: What's the difference between `em` and `rem`?**

A: "`em` is relative to the parent element's font size—can compound. `rem` is relative to the root (html) element's font size—consistent throughout. Use `rem` for predictable sizing."

**Q: What's the difference between `position: relative` and `position: absolute`?**

A: "Relative: positioned relative to its normal position, doesn't remove from flow. Absolute: positioned relative to nearest positioned ancestor, removed from flow. Absolute elements need a positioned parent (relative/absolute/fixed)."

---

*Next: [03 — Flexbox & Grid](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/FRONTEND/flexbox-grid.md)*
