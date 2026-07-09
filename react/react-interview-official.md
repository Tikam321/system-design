# React Interview Questions & Answers

A complete guide to React interview questions — covering fundamentals, JSX, components, hooks, state management, performance, routing, testing, and modern React (18/19) features.

---

## Table of Contents
1. [React Basics](#react-basics)
2. [JSX](#jsx)
3. [Components & Props](#components--props)
4. [State & Lifecycle](#state--lifecycle)
5. [Hooks](#hooks)
6. [Event Handling & Forms](#event-handling--forms)
7. [Rendering, Reconciliation & the Virtual DOM](#rendering-reconciliation--the-virtual-dom)
8. [Context API & State Management](#context-api--state-management)
9. [Performance Optimization](#performance-optimization)
10. [React Router](#react-router)
11. [Error Handling](#error-handling)
12. [Testing](#testing)
13. [Modern React (18/19) Features](#modern-react-1819-features)
14. [Common Coding Questions](#common-coding-questions)
15. [Advanced / Design Questions](#advanced--design-questions)

---

## React Basics

### 1. What is React? What problem does it solve?
React is a JavaScript library (not a full framework) for building user interfaces, primarily for single-page applications. It solves the problem of efficiently updating the UI when data changes, by using a component-based architecture and a virtual DOM to minimize expensive direct DOM manipulations.

### 2. What are the key features of React?
- **Component-based architecture** — UI split into reusable, isolated pieces.
- **Virtual DOM** — an in-memory representation of the UI for efficient diffing/updates.
- **Declarative UI** — you describe *what* the UI should look like for a given state, not *how* to update it step by step.
- **Unidirectional (one-way) data flow** — data flows from parent to child via props, making state changes predictable.
- **JSX** — a syntax extension letting you write HTML-like markup within JavaScript.

### 3. What is the difference between a library and a framework? Why is React called a library?
A **framework** dictates the overall structure of your application and calls your code (inversion of control) — e.g., Angular. A **library** is a focused tool you call into as needed, giving you more freedom over architecture. React is called a library because it focuses solely on the view layer — routing, state management, and other concerns are handled by separate libraries (React Router, Redux, etc.) that you choose and wire in yourself.

### 4. What is the Virtual DOM? How does it improve performance?
The Virtual DOM is a lightweight, in-memory JavaScript representation of the actual DOM. When state changes, React first updates the Virtual DOM, then compares (**diffs**) the new Virtual DOM tree against the previous one, computes the minimal set of actual DOM changes needed, and applies only those changes (batched) to the real DOM — avoiding costly, full re-renders of the entire page.

### 5. What is the difference between the real DOM and the virtual DOM?

| Real DOM | Virtual DOM |
|---|---|
| Direct manipulation is slow (triggers layout/reflow/repaint) | In-memory, manipulation is fast |
| Updates the entire tree/subtree even for small changes (without React) | Only the diffed changes are applied to the real DOM |
| Browser-native structure | React's own lightweight object representation |

### 6. What is reconciliation?
The algorithm React uses to compare the new Virtual DOM tree with the previous one (a "diffing" process) to determine the minimal number of real DOM mutations required to bring the DOM up to date with the latest state.

### 7. What is the React Fiber architecture?
Fiber is React's reconciliation engine (introduced in React 16), which re-implemented the core algorithm to support **incremental rendering** — splitting rendering work into units that can be paused, resumed, aborted, or prioritized, enabling features like concurrent rendering, Suspense, and better responsiveness on large UI updates.

### 8. What is the difference between React and ReactDOM?
`react` contains the core logic for defining components, managing state, and the reconciliation algorithm — it's platform agnostic. `react-dom` is the renderer that connects React to the actual browser DOM (for native mobile, `react-native` serves this role instead).

---

## JSX

### 9. What is JSX?
JSX (JavaScript XML) is a syntax extension that allows writing HTML-like markup directly within JavaScript code. It's not valid JavaScript by itself — it gets transpiled (via Babel) into `React.createElement()` calls.

```jsx
const element = <h1>Hello, world!</h1>;
// Transpiles roughly to:
const element = React.createElement('h1', null, 'Hello, world!');
```

### 10. Why can't browsers understand JSX directly?
Browsers only understand plain JavaScript. JSX must be compiled (typically by Babel, or the build tool's integrated compiler) into standard `React.createElement()` (or the newer JSX transform's `jsx()`) function calls before it can run.

### 11. What are the rules of JSX?
- Must return a **single root element** (or use a Fragment `<>...</>` to group multiple elements without an extra DOM node).
- All tags must be closed (`<img />`, not `<img>`).
- Use `className` instead of `class`, and `htmlFor` instead of `for` (since `class`/`for` are reserved JS keywords).
- JavaScript expressions are embedded using curly braces `{}`.
- Attribute names use camelCase (`onClick`, `tabIndex`).

### 12. What is a React Fragment? Why use it?
`<React.Fragment>` (or shorthand `<>...</>`) lets you group multiple children without introducing an extra wrapper node into the actual DOM — useful because JSX requires a single root but you don't always want an unnecessary `<div>` cluttering your markup/CSS layout.

### 13. How do you conditionally render JSX?
```jsx
// Ternary
{isLoggedIn ? <Dashboard /> : <Login />}

// Logical AND (short-circuit)
{hasError && <ErrorMessage />}

// Early return
if (!user) return <Spinner />;
return <Profile user={user} />;
```

---

## Components & Props

### 14. What is the difference between functional and class components?

| Functional Components | Class Components |
|---|---|
| Plain JavaScript functions returning JSX | ES6 classes extending `React.Component` |
| State/lifecycle via Hooks (`useState`, `useEffect`) | State via `this.state`, lifecycle methods (`componentDidMount`, etc.) |
| Simpler, less boilerplate | More verbose (`this` binding issues, constructors) |
| Modern standard approach (post Hooks, React 16.8+) | Legacy approach, still supported but rarely used for new code |

### 15. What are props? Are they mutable?
Props ("properties") are read-only inputs passed from a parent component to a child component. Props are **immutable** from the child's perspective — a component must never modify its own props; if the underlying data needs to change, that logic belongs in the parent (or shared state), which then passes new props down.

### 16. What is `props.children`?
A special prop that contains whatever is nested between a component's opening and closing tags in JSX, allowing components to act as generic wrappers/containers.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

<Card><p>Some content</p></Card>
```

### 17. What is prop drilling? How do you avoid it?
Prop drilling is passing data through multiple layers of nested components (via props) purely so a deeply nested child can access it, even though intermediate components don't use that data themselves. It's avoided using the **Context API**, or state management libraries (Redux, Zustand, Jotai), or component composition patterns.

### 18. What are default props, and how do you define them?
Values used for a prop when the parent doesn't explicitly pass one.

```jsx
function Button({ label = "Click me" }) {
  return <button>{label}</button>;
}
```
*(The old `Component.defaultProps = {...}` static property pattern for class/function components is being deprecated in favor of default parameter values.)*

### 19. What is PropTypes? Why use it (especially without TypeScript)?
`prop-types` is a library for runtime type-checking of props, throwing console warnings during development if a component receives props of the wrong type — a lightweight alternative to TypeScript for catching prop-related bugs early, though TypeScript is generally preferred in modern codebases for compile-time safety.

### 20. What is the difference between a controlled and an uncontrolled component?
- **Controlled**: form input's value is driven by React state — every keystroke updates state via `onChange`, and the input's `value` is set from that state (single source of truth in React).
- **Uncontrolled**: form input manages its own internal state in the DOM; React accesses its value on-demand via a `ref` instead of tracking every change.

```jsx
// Controlled
<input value={name} onChange={(e) => setName(e.target.value)} />

// Uncontrolled
<input ref={inputRef} defaultValue="initial" />
```

### 21. What is component composition? How does it differ from inheritance?
Composition means building complex UIs by combining smaller components together (nesting, passing `children`, render props) rather than using class inheritance to share behavior. React's official guidance strongly favors composition — inheritance hierarchies for UI components tend to become rigid and hard to reason about.

---

## State & Lifecycle

### 22. What is state in React?
Data that is local to a component and can change over time, triggering a re-render whenever it's updated. Unlike props, state is managed internally by the component itself (or lifted up to a parent/shared store).

### 23. What is the difference between state and props?

| State | Props |
|---|---|
| Managed within the component | Passed from parent to child |
| Mutable (via `setState`/`useState` setter) | Immutable from the child's perspective |
| Triggers re-render on change | Triggers re-render when parent passes new values |
| Local/private to the component | External data, "owned" by the parent |

### 24. What are the lifecycle methods in class components?
- **Mounting**: `constructor()` → `render()` → `componentDidMount()`
- **Updating**: `shouldComponentUpdate()` → `render()` → `componentDidUpdate()`
- **Unmounting**: `componentWillUnmount()`
- **Error handling**: `componentDidCatch()`, `static getDerivedStateFromError()`

### 25. How do you replicate lifecycle methods using hooks in functional components?
- `componentDidMount` → `useEffect(() => {...}, [])` (empty dependency array — runs once).
- `componentDidUpdate` → `useEffect(() => {...}, [dep1, dep2])` (runs when dependencies change).
- `componentWillUnmount` → the **cleanup function** returned from `useEffect`.

```jsx
useEffect(() => {
  const timer = setInterval(tick, 1000);
  return () => clearInterval(timer); // cleanup, like componentWillUnmount
}, []);
```

### 26. What is "lifting state up"?
Moving shared state from a child component up to their closest common ancestor, so that multiple sibling components can share and stay in sync with the same piece of state, passed back down via props.

### 27. Why shouldn't you mutate state directly?
Directly mutating state (e.g., `this.state.items.push(x)` or `state.count++`) doesn't trigger a re-render because React relies on detecting a *new* reference/value to know something changed — mutating in place leaves the reference identical, so React has no signal to update the UI. Always use the setter function (`setState`, or the updater from `useState`) with a new object/array.

```jsx
// Wrong
state.items.push(newItem);

// Right
setItems([...items, newItem]);
```

---

## Hooks

### 28. What are hooks? Why were they introduced?
Hooks (introduced in React 16.8) are functions that let functional components "hook into" React features like state and lifecycle, which were previously only available in class components. They were introduced to reduce the verbosity/confusion of class components (e.g., `this` binding, complex lifecycle splitting), and to allow easier reuse of stateful logic between components (via custom hooks) without patterns like render props or higher-order components.

### 29. What are the rules of hooks?
1. **Only call hooks at the top level** — never inside loops, conditions, or nested functions (ensures hooks are called in the same order on every render, which React relies on internally to associate state with the right hook call).
2. **Only call hooks from React function components or custom hooks** — not from regular JavaScript functions.

### 30. Explain `useState`.
Lets a functional component hold local state. Returns an array: the current state value, and a setter function to update it (triggering a re-render).

```jsx
const [count, setCount] = useState(0);
setCount(count + 1);
setCount(prev => prev + 1); // functional update, safer with stale closures
```

### 31. Why use the functional update form of `setState` (`setCount(prev => prev + 1)`)?
Because state updates may be batched/asynchronous, referencing the current `count` variable directly inside rapid successive calls can use a stale value from the render's closure. The functional form guarantees you're always operating on the most up-to-date state value at the time the update is actually applied.

### 32. Explain `useEffect`. What is the role of the dependency array?
Lets you perform side effects (data fetching, subscriptions, manual DOM manipulation, timers) in function components. The dependency array controls **when** the effect re-runs:
- `[]` (empty) → runs once after initial mount only.
- `[dep1, dep2]` → re-runs whenever any listed dependency changes.
- Omitted entirely → runs after **every** render.

```jsx
useEffect(() => {
  console.log('Effect ran because count changed:', count);
}, [count]);
```

### 33. What is the difference between `useEffect` and `useLayoutEffect`?
- `useEffect` runs **asynchronously**, after the browser has painted the screen — doesn't block visual updates.
- `useLayoutEffect` runs **synchronously**, immediately after DOM mutations but *before* the browser paints — used when you need to measure/adjust the DOM before the user sees anything (e.g., avoiding a visual flicker).

### 34. Explain `useContext`.
Lets a component read a value from the nearest matching `Context.Provider` up the tree, without needing to manually pass props through every intermediate level (avoiding prop drilling).

```jsx
const ThemeContext = createContext('light');

function Toolbar() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>...</div>;
}
```

### 35. Explain `useRef`. What are its common use cases?
Returns a mutable object (`{ current: value }`) that persists across renders **without** triggering a re-render when it changes. Common uses:
- Accessing/manipulating a DOM element directly (`inputRef.current.focus()`).
- Storing a mutable value that shouldn't cause re-renders (e.g., a timer ID, previous value tracking).

```jsx
const inputRef = useRef(null);
useEffect(() => { inputRef.current.focus(); }, []);
<input ref={inputRef} />
```

### 36. Difference between `useRef` and `useState`?

| `useRef` | `useState` |
|---|---|
| Mutating `.current` does NOT trigger re-render | Calling the setter DOES trigger re-render |
| Value persists across renders | Value persists across renders |
| Good for values outside the render/UI cycle (DOM nodes, timers) | Good for values that should drive what's rendered |

### 37. Explain `useMemo`. When should you use it?
Memoizes the **result** of an expensive computation, recomputing it only when one of its dependencies changes — avoiding unnecessary recalculation on every render.

```jsx
const sortedList = useMemo(() => expensiveSort(list), [list]);
```
Use it when a calculation is genuinely expensive and its inputs don't change every render — overusing `useMemo` for trivial computations adds unnecessary complexity/overhead.

### 38. Explain `useCallback`. How does it differ from `useMemo`?
`useCallback` memoizes a **function reference** itself (rather than a computed value), preventing a new function instance from being created on every render — important when passing callbacks to memoized child components (`React.memo`) to avoid breaking their memoization due to a "new" prop reference each time.

```jsx
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```
`useMemo(() => fn, deps)` and `useCallback(fn, deps)` are functionally equivalent — `useCallback(fn, deps)` is just shorthand for `useMemo(() => fn, deps)`.

### 39. Explain `useReducer`. When would you prefer it over `useState`?
An alternative to `useState` for managing more complex state logic, especially when the next state depends on the previous state in non-trivial ways, or when multiple sub-values need to be updated together via distinct "action" types (similar to Redux's reducer pattern).

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    default: throw new Error();
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0 });
dispatch({ type: 'increment' });
```

### 40. What are custom hooks? Why create them?
Custom hooks are plain JavaScript functions (by convention prefixed with `use`) that call other hooks internally, allowing you to extract and reuse stateful logic across multiple components without duplicating code or resorting to render props/HOCs.

```jsx
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  useEffect(() => {
    const handler = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler);
  }, []);
  return width;
}
```

### 41. What is the `useImperativeHandle` hook used for?
Used together with `forwardRef` to customize the instance value exposed to a parent when using `ref` on a child component — letting you expose only specific methods/values instead of the entire underlying DOM node or internal implementation.

### 42. What is `useId` (React 18)? Why was it introduced?
Generates a unique, stable ID string that's consistent between server and client rendering — needed because using something like `Math.random()` for IDs (e.g., for accessibility attributes like `aria-describedby`) would produce mismatches between server-rendered and client-rendered HTML.

### 43. What is `useTransition` and `useDeferredValue` (React 18 concurrent features)?
- `useTransition` lets you mark certain state updates as **non-urgent ("transitions")**, so React can prioritize more urgent updates (like typing in an input) and delay/interrupt the transition's rendering without blocking the UI.
- `useDeferredValue` lets you defer re-rendering a non-critical part of the UI until more urgent updates have been processed, effectively "debouncing" a value's use in rendering during heavy updates.

```jsx
const [isPending, startTransition] = useTransition();
startTransition(() => {
  setSearchResults(filterResults(query));
});
```

---

## Event Handling & Forms

### 44. How does event handling differ in React compared to plain DOM?
React uses **SyntheticEvent**, a cross-browser wrapper around the native event object, providing consistent behavior across browsers. Event handlers are passed as props using camelCase (`onClick`, not `onclick`), and you pass a function reference rather than a string.

```jsx
<button onClick={handleClick}>Click</button> // not onClick="handleClick()"
```

### 45. What is a SyntheticEvent? Why does React use it?
A cross-browser wrapper object around the browser's native event, normalizing differences between browser implementations, and (historically) enabling event pooling for performance. As of React 17+, event pooling was removed, so you no longer need to call `event.persist()` to access event properties asynchronously.

### 46. How do you handle form submission in React?
```jsx
function Form() {
  const [value, setValue] = useState('');

  function handleSubmit(e) {
    e.preventDefault(); // prevent default full-page reload
    console.log('Submitted:', value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### 47. How do you pass arguments to an event handler?
```jsx
<button onClick={() => handleDelete(item.id)}>Delete</button>
// Avoid handleDelete(item.id) directly — that would invoke it immediately during render
```

---

## Rendering, Reconciliation & the Virtual DOM

### 48. What are keys in React lists? Why are they important?
`key` is a special prop that helps React identify which items in a list have changed, been added, or removed, enabling efficient re-use of existing DOM elements/components instead of destroying and recreating them during reconciliation.

```jsx
{items.map(item => <li key={item.id}>{item.name}</li>)}
```

### 49. Why shouldn't you use array index as a key?
If the list order can change (items inserted, removed, or reordered), using the index as a key means React may incorrectly match items to the wrong DOM elements/state across re-renders — causing bugs like incorrect component state being preserved for the wrong item, or unnecessary re-renders/remounts. Index keys are only safe for lists that are static and never reordered/filtered.

### 50. What happens when you call `setState`? Is it synchronous or asynchronous?
Calling the state setter schedules a re-render — it does **not** immediately mutate the state variable in place. In React 18+, state updates are **batched** by default across event handlers, timeouts, promises, and native event handlers (previously batching only happened within React event handlers), meaning multiple `setState` calls within the same tick are grouped into a single re-render for performance.

### 51. What is the difference between `React.memo`, `useMemo`, and `useCallback`?
- `React.memo(Component)` — a higher-order component that memoizes an entire **component**, skipping re-render if its props haven't changed (shallow comparison).
- `useMemo` — memoizes a computed **value**.
- `useCallback` — memoizes a **function reference**.

### 52. What causes a component to re-render?
1. Its own state changes (`setState`/hook setter called).
2. Its parent re-renders (by default, all children re-render too, unless wrapped in `React.memo`).
3. Context value it consumes changes.
4. Force update (rarely used directly).

### 53. What is the difference between shallow comparison and deep comparison in the context of `React.memo`?
`React.memo`'s default comparison checks if each prop is `===` equal to its previous value (**shallow** comparison) — for objects/arrays/functions, this means a *new* reference (even with identical contents) will be considered "different," triggering a re-render. You can pass a custom comparison function as `React.memo`'s second argument for deep/custom equality checks.

---

## Context API & State Management

### 54. What is the Context API? What problem does it solve?
A built-in React feature for sharing values (theme, authenticated user, locale, etc.) across a component tree without manually passing props through every intermediate level — solving the prop-drilling problem for globally relevant data.

```jsx
const UserContext = createContext(null);

function App() {
  return (
    <UserContext.Provider value={{ name: "Alice" }}>
      <Dashboard />
    </UserContext.Provider>
  );
}

function Dashboard() {
  const user = useContext(UserContext);
  return <p>Welcome, {user.name}</p>;
}
```

### 55. What are the downsides of Context API for state management?
- Any component consuming a context re-renders whenever the context value changes, even if it only cares about part of that value — can cause unnecessary re-renders if not split carefully.
- Not optimized for very frequent, high-velocity updates (e.g., real-time collaborative state) — a dedicated state library performs better there.
- No built-in dev tools/middleware/time-travel debugging, unlike Redux.

### 56. Compare Context API vs Redux vs Zustand/Jotai for state management.

| | Context API | Redux | Zustand/Jotai |
|---|---|---|---|
| Built-in | Yes | No (external library) | No (external library) |
| Boilerplate | Low | Historically high (though Redux Toolkit reduces this significantly) | Very low |
| DevTools/middleware | No | Excellent (Redux DevTools, time-travel) | Basic/growing support |
| Best for | Simple/global, infrequently-changing state (theme, auth) | Large apps needing predictable state, strict data flow, complex middleware | Simpler global state without Redux's ceremony |

### 57. What is Redux? Explain its core principles.
A predictable state container based on three principles:
1. **Single source of truth** — the entire application state lives in one store.
2. **State is read-only** — the only way to change state is by dispatching an **action** (a plain object describing what happened).
3. **Changes are made with pure functions** — **reducers** take the previous state and an action, and return a new state, without mutating the original.

```jsx
function counterReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    default: return state;
  }
}
```

### 58. What are the differences between Redux and the Flux architecture?
Flux is the original architectural pattern (from Facebook) with a unidirectional data flow: Action → Dispatcher → Store → View. Redux is a specific, simplified implementation of Flux's ideas — it removes the separate "Dispatcher" concept, uses pure reducer functions, and enforces a single store instead of Flux's multiple stores.

---

## Performance Optimization

### 59. What are common causes of unnecessary re-renders in React?
- Passing new object/array/function literals as props on every render (breaks `React.memo`/shallow comparisons).
- Not memoizing expensive computations (`useMemo`) or callbacks (`useCallback`).
- Context value changing, causing all consumers to re-render.
- Not using keys correctly in lists.
- Overly broad state — updating a large shared state object when only a small part actually changed.

### 60. What is code-splitting? How do you implement it in React?
Splitting your JavaScript bundle into smaller chunks that are loaded on-demand (e.g., per route) rather than all at once, reducing initial load time. Implemented via dynamic `import()` combined with `React.lazy()` and `Suspense`.

```jsx
const Dashboard = React.lazy(() => import('./Dashboard'));

<Suspense fallback={<Spinner />}>
  <Dashboard />
</Suspense>
```

### 61. What is `React.lazy` and `Suspense`?
`React.lazy()` lets you dynamically import a component so its code is only fetched when it's actually needed. `<Suspense>` lets you specify a fallback UI (like a loading spinner) to show while the lazy component (or any async operation wrapped in Suspense, like data fetching in newer React versions) is loading.

### 62. What is windowing/virtualization? Why is it useful?
A technique (via libraries like `react-window` or `react-virtualized`) that renders only the visible items of a long list (plus a small buffer), rather than the entire list — drastically reducing the number of DOM nodes and improving performance for very large lists/tables.

### 63. How would you profile and debug performance issues in a React app?
- Use **React DevTools Profiler** to record renders and see which components re-rendered and why, and how long each took.
- Check for missing `key`s or unnecessary prop object/array recreation.
- Use `why-did-you-render` (a debug library) to catch avoidable re-renders.
- Use browser Performance tab / Lighthouse for broader page performance metrics.

---

## React Router

### 64. What is React Router? Why is it needed?
A standard library for handling client-side routing in single-page React applications — mapping URL paths to components, enabling navigation without a full page reload, and syncing the browser's URL/history with the rendered UI.

### 65. What is the difference between `BrowserRouter` and `HashRouter`?
- `BrowserRouter` uses the HTML5 History API for clean URLs (`/about`), requires server-side configuration to handle direct navigation to sub-routes.
- `HashRouter` uses the URL hash (`/#/about`) — works without any server configuration since the hash portion is never sent to the server, but produces less clean URLs.

### 66. How do you implement nested routes and dynamic route parameters?
```jsx
<Routes>
  <Route path="/users/:userId" element={<UserProfile />} />
</Routes>

function UserProfile() {
  const { userId } = useParams();
  return <p>User ID: {userId}</p>;
}
```

### 67. How do you implement protected/private routes?
```jsx
function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuth();
  return isAuthenticated ? children : <Navigate to="/login" />;
}

<Route path="/dashboard" element={
  <ProtectedRoute><Dashboard /></ProtectedRoute>
} />
```

---

## Error Handling

### 68. What are Error Boundaries? How do you implement one?
Class components (there's no hook equivalent as of now) that catch JavaScript errors anywhere in their child component tree during rendering, in lifecycle methods, and in constructors — logging the error and displaying a fallback UI instead of crashing the whole app.

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(error, info) { console.error(error, info); }
  render() {
    if (this.state.hasError) return <h1>Something went wrong.</h1>;
    return this.props.children;
  }
}
```

### 69. What errors are NOT caught by Error Boundaries?
- Errors inside event handlers (use regular `try/catch` there instead).
- Errors in asynchronous code (`setTimeout`, promises) outside of rendering.
- Errors during server-side rendering.
- Errors thrown in the error boundary component itself.

---

## Testing

### 70. What testing tools are commonly used with React?
- **Jest** — test runner and assertion library.
- **React Testing Library (RTL)** — encourages testing components the way a user would interact with them (querying by visible text/roles rather than internal implementation details).
- **Enzyme** — an older alternative (less favored now) that allows shallow rendering and inspecting component internals directly.
- **Cypress/Playwright** — end-to-end testing tools that test the full running application in a real browser.

### 71. What is the philosophy behind React Testing Library vs Enzyme?
RTL's guiding principle is: *"The more your tests resemble the way your software is used, the more confidence they can give you."* It discourages testing implementation details (internal state, private methods) in favor of testing observable behavior (what's rendered, what happens when a user clicks a button) — making tests more resilient to refactors.

```jsx
test('renders greeting after button click', () => {
  render(<Greeting />);
  fireEvent.click(screen.getByRole('button', { name: /say hello/i }));
  expect(screen.getByText(/hello!/i)).toBeInTheDocument();
});
```

### 72. What is snapshot testing?
A testing technique where a component's rendered output is serialized and saved as a "snapshot" file; future test runs compare the current output against the saved snapshot, flagging any unexpected changes. Useful for catching unintended UI regressions, though overuse can lead to noisy, low-value tests if snapshots aren't reviewed carefully.

---

## Modern React (18/19) Features

### 73. What is Concurrent Rendering in React 18?
A set of new capabilities enabled by React's Fiber architecture that let React prepare multiple versions of the UI at the same time, interrupt rendering to handle more urgent updates, and avoid blocking the main thread on long renders — enabling smoother, more responsive UIs (powers `useTransition`, `useDeferredValue`, and Suspense improvements).

### 74. What changed with `createRoot` in React 18?
React 18 introduced the new root API (`ReactDOM.createRoot(container).render(<App />)`), replacing the legacy `ReactDOM.render()`. This opts the app into concurrent features — without switching to `createRoot`, new concurrent capabilities aren't enabled even on React 18.

### 75. What is automatic batching in React 18?
Prior to React 18, state updates were only batched (grouped into a single re-render) inside React event handlers. React 18 extends batching to **all** updates — including those inside promises, `setTimeout`, and native event handlers — improving performance by default.

### 76. What are Server Components (React Server Components / RSC)?
A component type (used notably in frameworks like Next.js App Router) that renders entirely on the server, sending only the resulting output (not the component's JS bundle) to the client — reducing client-side JavaScript bundle size, and allowing direct server-side data access (e.g., database queries) within the component itself, without a separate API layer. They can't use hooks like `useState`/`useEffect` or browser-only APIs, since they never run on the client.

### 77. What is the difference between Server Components and Client Components?

| Server Components | Client Components |
|---|---|
| Render on the server only | Render on both server (initial HTML) and client (hydration) |
| No hooks/state/event handlers | Full access to hooks, state, browser APIs |
| Zero JS shipped to the browser for that component | JS bundle shipped and hydrated |
| Marked by default (no directive) in RSC-supporting frameworks | Marked with `"use client"` directive |

### 78. What is the `use` hook (React 19)?
A new hook that lets you read the value of a resolved Promise or Context directly within rendering (including conditionally, unlike other hooks) — commonly used together with Suspense to handle async data fetching more directly within components.

### 79. What are Actions and `useActionState` / `useFormStatus` (React 19)?
React 19 introduces first-class support for handling async form submissions ("Actions") — functions that can be passed directly to a form's `action` prop, automatically managing pending states, errors, and optimistic updates, reducing the boilerplate previously needed with manual `useState`/`useEffect` combinations for form submission handling.

---

## Common Coding Questions

### 80. Build a simple counter component.
```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <button onClick={() => setCount(c => c - 1)}>-</button>
    </div>
  );
}
```

### 81. Build a debounced search input.
```jsx
function SearchInput({ onSearch }) {
  const [query, setQuery] = useState('');

  useEffect(() => {
    const timer = setTimeout(() => onSearch(query), 500);
    return () => clearTimeout(timer); // cancel previous timer on each keystroke
  }, [query, onSearch]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

### 82. Implement a custom `useFetch` hook.
```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(json => { if (!cancelled) { setData(json); setLoading(false); } })
      .catch(err => { if (!cancelled) { setError(err); setLoading(false); } });
    return () => { cancelled = true; }; // avoid setting state on unmounted component
  }, [url]);

  return { data, loading, error };
}
```

### 83. Implement an accordion/toggle component.
```jsx
function Accordion({ items }) {
  const [openIndex, setOpenIndex] = useState(null);
  return (
    <div>
      {items.map((item, i) => (
        <div key={item.id}>
          <button onClick={() => setOpenIndex(openIndex === i ? null : i)}>
            {item.title}
          </button>
          {openIndex === i && <p>{item.content}</p>}
        </div>
      ))}
    </div>
  );
}
```

### 84. Implement a simple modal using a portal.
```jsx
function Modal({ children, onClose }) {
  return ReactDOM.createPortal(
    <div className="overlay" onClick={onClose}>
      <div className="modal" onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    document.getElementById('modal-root')
  );
}
```

### 85. Fetch and display a paginated list.
```jsx
function PaginatedList() {
  const [page, setPage] = useState(1);
  const { data, loading } = useFetch(`/api/items?page=${page}`);

  return (
    <div>
      {loading ? <p>Loading...</p> : data?.items.map(item => <p key={item.id}>{item.name}</p>)}
      <button disabled={page === 1} onClick={() => setPage(p => p - 1)}>Prev</button>
      <button onClick={() => setPage(p => p + 1)}>Next</button>
    </div>
  );
}
```

---

## Advanced / Design Questions

### 86. What is a Higher-Order Component (HOC)?
A function that takes a component and returns a new, enhanced component — a pattern for reusing component logic (e.g., adding authentication checks, injecting extra props). Largely superseded by custom hooks in modern React, but still seen in some libraries.

```jsx
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { isAuthenticated } = useAuth();
    if (!isAuthenticated) return <Redirect to="/login" />;
    return <WrappedComponent {...props} />;
  };
}
```

### 87. What is the "render props" pattern?
A pattern where a component takes a function as a prop (often literally named `render` or passed as `children`) and calls it to determine what to render — allowing logic sharing between components, similar in purpose to HOCs and custom hooks, though largely replaced by hooks in modern codebases.

```jsx
<DataProvider render={data => <Chart data={data} />} />
```

### 88. What is the compound components pattern?
A pattern where multiple components work together to form a cohesive API, implicitly sharing state via Context, letting consumers compose the pieces flexibly (e.g., `<Tabs><Tabs.List><Tabs.Tab /></Tabs.List><Tabs.Panel /></Tabs>`), commonly used in design system/component libraries.

### 89. How would you architect a large-scale React application (folder structure, state management, data fetching)?
Common approaches include:
- **Feature-based folder structure** (grouping by domain/feature rather than by file type) for better scalability than a flat `components/`, `hooks/`, `utils/` split.
- **Separating server state from client/UI state** — using a dedicated data-fetching library (React Query/TanStack Query, SWR) for server state (caching, revalidation, retries) rather than shoving API data into Redux/Context.
- **Using Context or a lightweight store (Zustand/Jotai) for global UI state**, reserving Redux for genuinely complex, cross-cutting state needs.
- **Design system/component library** for consistent, reusable UI primitives.
- **Code-splitting by route** to keep initial bundle size manageable.

### 90. What is hydration in the context of server-side rendering (SSR)?
The process where React "attaches" event listeners and internal state to server-rendered static HTML on the client, making the already-visible markup interactive — without needing to re-render/re-create the DOM from scratch, since the DOM already exists from the server's HTML output.

### 91. What are the trade-offs of SSR vs CSR (Client-Side Rendering) vs SSG (Static Site Generation)?

| | CSR | SSR | SSG |
|---|---|---|---|
| Initial load | Slower (blank page until JS loads/executes) | Faster (HTML ready immediately) | Fastest (pre-built HTML served from CDN) |
| SEO | Weaker (unless pre-rendered) | Strong | Strong |
| Server load | Low (just serves static JS bundle) | Higher (renders on every request) | Very low (pre-built at build time) |
| Data freshness | Always fresh (fetched client-side) | Fresh (rendered per request) | Can be stale (unless combined with ISR/revalidation) |
| Best for | Highly interactive apps/dashboards behind auth | Content that needs to be fresh and SEO-friendly | Blogs, marketing pages, docs — content that changes infrequently |

### 92. What is the difference between `key` and `ref`? Are they accessible as regular props?
Both `key` and `ref` are special props reserved by React for internal purposes (list reconciliation and imperative DOM/instance access, respectively) — they are **not** accessible via `props.key` or `props.ref` inside the component; React strips them out before passing props through.

### 93. How does React handle synthetic event delegation internally?
Rather than attaching a listener to every individual DOM element, React (historically) attaches a single listener at the root of the document (or since React 17, at the root DOM container the app is rendered into) and uses event bubbling to determine which component's handler should fire — improving memory efficiency and simplifying dynamic listener management.

---

## Quick Cheat-Sheet Summary

| Concept | Key Point |
|---|---|
| Virtual DOM | In-memory diffing to minimize real DOM updates |
| JSX | Syntactic sugar for `React.createElement()` calls |
| Props vs State | Props = external, immutable; State = internal, mutable |
| `useEffect` deps | `[]` = once, `[x]` = on x change, omitted = every render |
| `useMemo` vs `useCallback` | Memoize a value vs memoize a function reference |
| Keys in lists | Use stable unique IDs, not array index (if list order can change) |
| Context API | Solves prop drilling; re-renders all consumers on value change |
| `React.memo` | Skips re-render if props are shallowly equal |
| Error Boundaries | Class-only; doesn't catch async/event handler errors |
| Concurrent rendering | Enables `useTransition`, `useDeferredValue`, better responsiveness |
| Server Components | Zero client JS, no hooks, server-only data access |

---

*Good luck with your interview! Be ready to explain trade-offs (e.g., "when would you NOT use `useMemo`?") — interviewers often probe for judgment, not just definitions.*
