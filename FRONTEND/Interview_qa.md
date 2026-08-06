# 12 — Frontend Interview Questions & Answers

## Real Questions, Real Answers

---

### HTML & CSS Questions

#### Q: What's the CSS box model?

**A:** "Every element is a rectangular box with content, padding, border, and margin. With box-sizing: border-box, width includes padding and border. With content-box (default), width is just content."

---

#### Q: How does CSS specificity work?

**A:** "Inline styles (1000) > IDs (100) > classes/attributes/pseudo-classes (10) > elements/pseudo-elements (1). !important overrides everything but should be avoided."

---

#### Q: What's the difference between Flexbox and Grid?

**A:** "Flexbox is one-dimensional (row OR column). Grid is two-dimensional (rows AND columns). Use Flexbox for components (navbars), Grid for page layouts (dashboards)."

---

### JavaScript Questions

#### Q: What is a closure?

**A:** "A function that remembers variables from its outer scope even after the outer function returns. Used for data privacy, function factories, maintaining state in event handlers."

---

#### Q: Explain the event loop.

**A:** "JavaScript is single-threaded. The event loop processes: (1) Call stack (synchronous), (2) Microtask queue (Promises), (3) Macrotask queue (setTimeout). Microtasks have priority."

---

#### Q: What's the difference between `==` and `===`?

**A:** "`==` performs type coercion (5 == '5' is true). `===` compares without coercion (5 === '5' is false). Always use `===`."

---

#### Q: What's the difference between `var`, `let`, and `const`?

**A:** "`var`: function-scoped, hoisted. `let`: block-scoped, not hoisted. `const`: block-scoped, can't be reassigned. Use `const` by default, `let` when reassignment needed, avoid `var`."

---

#### Q: What are Promises?

**A:** "Objects representing eventual completion/failure of async operations. Chain with .then()/.catch(). Async/await is syntactic sugar over Promises."

---

#### Q: What's `this` in JavaScript?

**A:** "Depends on how function is called: method → object, standalone → window/undefined, arrow → inherited, constructor → new instance, event handler → element."

---

### React Questions

#### Q: What's the virtual DOM?

**A:** "In-memory representation of real DOM. When state changes, React creates new virtual DOM, diffs with previous, updates only changed parts in real DOM. Faster than direct DOM manipulation."

---

#### Q: What's the difference between `useMemo` and `useCallback`?

**A:** "useMemo memoizes a VALUE (computation result). useCallback memoizes a FUNCTION (reference). Both prevent unnecessary re-computation/re-renders."

---

#### Q: When does a React component re-render?

**A:** "When state changes, props change, parent re-renders, or context changes. Prevent with React.memo, useMemo, useCallback."

---

#### Q: What are React hooks?

**A:** "Functions that let you use state and lifecycle in function components. useState for state, useEffect for side effects, useRef for refs, useContext for context, useMemo/useCallback for optimization."

---

#### Q: What's the difference between SSR and SSG?

**A:** "SSR: HTML generated per request—dynamic content. SSG: HTML generated at build time—static content, faster. ISR: SSG with revalidation."

---

### TypeScript Questions

#### Q: What's the difference between `interface` and `type`?

**A:** "Interfaces: extendable, declaration merging, best for objects. Types: can do unions, intersections, primitives. Use interfaces for object shapes; types for complex types."

---

#### Q: What are generics?

**A:** "Type variables for reusable components. function identity<T>(value: T): T. Specified when calling or inferred. Used for type-safe collections, API responses."

---

### System Design Questions

#### Q: How would you optimize a slow web application?

**A:** "1) Code splitting and lazy loading. 2) Image optimization (WebP, srcset). 3) Caching (CDN, browser cache). 4) Minimize bundle size (tree shaking). 5) Virtual scrolling for large lists. 6) Debounce expensive operations. 7) Memoize components."

---

#### Q: How do you handle state management in React?

**A:** "useState for local state. useContext for shared state. useReducer for complex logic. Redux/Zustand for global state. Server state with React Query/SWR."

---

### Quick Reference

| Topic | Key Points |
|-------|-----------|
| Closure | Function + outer scope variables |
| Event Loop | Call stack → Microtasks → Macrotasks |
| Promises | Async values, chainable, async/await |
| this | Depends on call context |
| Virtual DOM | Diffing + reconciliation |
| Hooks | useState, useEffect, useRef, useMemo |
| SSR | Server renders HTML per request |
| SSG | HTML built at build time |
| TypeScript | Static types, generics, utility types |

---

*Good luck with your frontend interviews!*
