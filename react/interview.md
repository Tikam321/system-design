1.Custom react hook example 

Yes. Here’s a simple custom hook + component usage example.

```java
// useCounter.js
import { useState } from "react";

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount((c) => c + 1);
  const decrement = () => setCount((c) => c - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}
// Counter.jsx
import React from "react";
import { useCounter } from "./useCounter";

export default function Counter() {
  const { count, increment, decrement, reset } = useCounter(5);

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```
A custom hook is just a reusable func


2. what is useImperative is used for ?
```java
import React, { useRef, useImperativeHandle, forwardRef } from "react";

const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focusInput() {
      inputRef.current.focus();
    },
    clear() {
      inputRef.current.value = "";
    }
  }));

  return <input ref={inputRef} />;
});

export default function Parent() {
  const ref = useRef();

  return (
    <>
      <CustomInput ref={ref} />
      <button onClick={() => ref.current.focusInput()}>Focus</button>
      <button onClick={() => ref.current.clear()}>Clear</button>
    </>
  );
}
```
- Parent can access only the methods/properties that child exposes through useImperativeHandle (via ref), for example focusInput() or clear().
- So it is controlled access, not full direct access to all child internals.

3. what is reducer give an example ?

```java
import React, { useReducer } from "react";

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    case "reset":
      return initialState;
    default:
      return state;
  }
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <h2>Count: {state.count}</h2>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "reset" })}>Reset</button>
    </div>
  );
}
```

4. how do you cancel the async operation in useEffect ?
- Here, the cleanup function runs when the component unmounts. If the request finishes later, the state update is skipped.
```java

useEffect(() => {
  let isMounted = true;
  fetch("/api/data")
    .then(res => res.json())
    .then(data => {
      if (isMounted) {
        setData(data);
      }
    });
  return () => {
    isMounted = false;
  };
}, []);
```

5. What is the difference between useMemo and useCallback?
Both useMemo and useCallback are performance optimization hooks, but they optimize different things.
useMemo is used to memoize a value, while useCallback is used to memoize a function.
useMemo: memoizing a calculated value
Use useMemo when you have an expensive calculation and don’t want it to run on every render.
```java
const expensiveValue = useMemo(() => {
  return items.reduce((total, item) => total + item.price, 0);
}, [items]);
```
Here, the calculation only runs when items change, not on every render.

useCallback: memoizing a function

Use useCallback when you pass a function to a child component and want to avoid unnecessary re-renders.
```java
const handleClick = useCallback(() => {
  setCount(c => c + 1);
}, []);
```
Without useCallback, a new function is created on every render, which can cause memoized child components to re-render unnecessarily.

6. Why can useEffect(() => {}, []) still run twice in development?
At first glance, this feels confusing because an empty dependency array means the effect should run only once. And in production, it does.

The reason it can run twice in development is React Strict Mode.

In React 18, Strict Mode intentionally runs certain lifecycle logic twice in development only to help developers catch bugs early. This includes mounting, unmounting, and re-running effects. The idea is to expose side effects that are not properly cleaned up.

Here’s an example:
```java
useEffect(() => {
  console.log("Effect ran");

  return () => {
    console.log("Cleanup");
  };
}, []);
```
In development, you might see:

Effect ran

Cleanup

Effect ran

This happens because React:

Mounts the component
Runs the effect
Immediately cleans it up
Mounts it again and runs the effect again
This does not happen in production, and it’s not a bug.

The key takeaway is that effects should always be written in a way that they can safely run more than once. If your effect causes issues when run twice, it usually means cleanup logic is missing or incomplete.

7. How do you implement route-level code splitting (lazy loading routes)?
Route-level code splitting means loading a page only when the user visits that route, instead of loading all pages at once. This helps reduce the initial bundle size and makes the app load faster.

- In React, this is done using React.lazy along with Suspense.
- const Dashboard = React.lazy(() => import("./Dashboard"));
- const Profile = React.lazy(() => import("./Profile"));
- These components are not loaded immediately. They are fetched only when React needs to render them.

You then wrap your routes with Suspense to show a fallback UI while the component is loading.
```java
<Suspense fallback={<div>Loading...</div>}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/profile" element={<Profile />} />
  </Routes>
</Suspense>
```
When the user navigates to /dashboard, React downloads the Dashboard code on demand. Until then, the fallback UI is shown.

So basically, lazy loading routes improves performance by loading pages only when they are actually needed.

8. What is reconciliation in React and how does it relate to keys?
Reconciliation is the process React uses to decide what needs to change in the DOM when state or props update.

When a component re-renders, React creates a new virtual tree and compares it with the previous one. Instead of updating everything, React figures out the minimum number of changes needed to update the UI.

This is where keys are used.

When rendering lists, React uses keys to understand which item is which between renders.

```java
{items.map(item => (
  <ListItem key={item.id} data={item} />
))}
```
Keys help React match old elements with new ones. If keys are stable, React can:

Reuse existing DOM elements
Preserve state
Avoid unnecessary re-mounts
If keys are missing or unstable, React may:

Reorder elements incorrectly
Lose component state
Cause UI glitches like input focus loss
This is why using array index as a key often leads to bugs when items are added, removed, or reordered.


9. Why does console.log(state) show the old value right after setState?
This happens because state updates in React are asynchronous.

When you call setState, React does not update the state immediately. Instead, it schedules the update and re-renders the component later. So if you log the state right after calling setState, you’re still seeing the previous value.

setCount(count + 1);
console.log(count); // logs old value
Here, count hasn’t updated yet because the component hasn’t re-rendered.

If you want to work with the updated value, you should either:

Use the updated state during the next render
 
Or use a useEffect that runs when the state changes
useEffect(() => {
  console.log(count); // logs updated value
}, [count]);
Another safe approach is using a functional update when the new state depends on the previous one:

10. Why do components re-render even when props “look the same”?
A component can re-render even if props appear the same because React checks references, not deep equality.

For primitive values like numbers or strings, this usually isn’t an issue. But for objects, arrays, and functions, a new reference is created on every render.

<Child user={{ name: "John" }} />
Even though the object looks the same, { name: "John" } is a new object each time, so React treats it as changed and re-renders the child.


11. Why do components re-render even when props “look the same”?
A component can re-render even if props appear the same because React checks references, not deep equality.
For primitive values like numbers or strings, this usually isn’t an issue. But for objects, arrays, and functions, a new reference is created on every render.
<Child user={{ name: "John" }} />
Even though the object looks the same, { name: "John" } is a new object each time, so React treats it as changed and re-renders the child.
The same applies to functions:
<Child onClick={() => handleClick()} />
A new function is created on every render, which causes re-renders even if the logic is unchanged.
To avoid unnecessary re-renders, you can memoize values and functions using useMemo and useCallback.
const user = useMemo(() => ({ name: "John" }), []);
const handleClick = useCallback(() => {}, []);
You can also wrap child components with React.memo so they only re-render when props actually change.

In short, components re-render because props are new references, not because the values look different.

12. What is the difference between re-render and re-mount in React?
- A re-render happens when a component updates because its state, props, or context change. The component function runs again, but React keeps the existing component instance.
During a re-render:
State is preserved
DOM elements may be updated
useEffect runs again only if dependencies change
setCount(count + 1); // causes re-render
A re-mount, on the other hand, means the component is destroyed and created again from scratch.
During a re-mount:
State is reset
Effects are cleaned up and re-run
The component behaves like it’s rendering for the first time
Re-mounting usually happens when:
A component’s key changes
Conditional rendering removes and re-adds a component
React Strict Mode (development only) intentionally remounts components


12. Why does using array index as key cause UI bugs?
React uses key to understand which list item is which between renders. When you use the array index as a key, that identity can break when the list changes.
Here’s a common example:
{items.map((item, index) => (
  <input key={index} defaultValue={item.name} />
))}
Now imagine this list:
["Apple", "Banana", "Cherry"]
If you remove "Apple", the list becomes:
["Banana", "Cherry"]
React sees:
index 0 - was Apple, now Banana
index 1 - was Banana, now Cherry
Because the keys are reused, React reuses the wrong DOM elements. This can cause issues like:
Input values appearing in the wrong row
Focus jumping unexpectedly
Animations behaving incorrectly
The UI looks buggy even though the data is correct.

This doesn’t happen when you use a stable, unique key, like an ID:

<input key={item.id} />
Using array index as a key is only safe when:

The list never changes
Items are never added, removed, or reordered
In applications, those conditions are rare, which is why index-based keys often lead to subtle UI bugs.

13. What causes an infinite re-render loop and how do you debug it?
An infinite re-render loop happens when a component keeps triggering its own re-render without stopping. This usually occurs when state is updated during rendering or on every render cycle.
A very common cause is calling a state update directly inside the component body.
function Component() {
  const [count, setCount] = useState(0);
  setCount(count + 1);
  return <div>{count}</div>;
}
Here, every render calls setCount, which triggers another render, and the cycle never ends.
Another frequent cause is using useEffect incorrectly. If an effect updates state but does not have a proper dependency array, it will run after every render.
useEffect(() => {
  setCount(count + 1);
});
Since the effect runs after each render and updates state again, it creates a loop.

14. Why does React.memo sometimes not prevent re-renders?
React.memo prevents re-renders only when props remain referentially the same between renders. React does not compare values deeply; it only checks whether the references have changed.

This becomes an issue when props are objects, arrays, or functions.

const Child = React.memo(({ user }) => {
  return <div>{user.name}</div>;
});
<Child user={{ name: "John" }} />
Even though the object looks the same, a new object is created on every render. Because the reference changes, React.memo allows the re-render.

The same issue happens with functions passed as props.

<Child onClick={() => handleClick()} />
Each render creates a new function reference, so the child component re-renders.

To make React.memo effective, props should be stabilized using useMemo and useCallback.

const user = useMemo(() => ({ name: "John" }), []);
const handleClick = useCallback(() => {}, []);
It is also important to remember that React.memo only checks props. If a component uses its own state or consumes context, it will still re-render when those values change.

15. What happens when you update state based on previous state incorrectly?
When you update state using the current state value directly, React may use an outdated value, especially when updates are batched or triggered multiple times quickly.

For example:
setCount(count + 1);
setCount(count + 1);
You might expect the count to increase by 2, but it usually increases by only 1. This happens because both updates read the same old value of count before React applies the update.
The correct way to update state based on the previous value is to use a functional state update.
setCount(prevCount => prevCount + 1);
setCount(prevCount => prevCount + 1);
Here, each update receives the latest state value, so the final result is correct.
Incorrect state updates can lead to:
Lost updates
Inconsistent UI
Bugs that appear only under certain conditions
Any time the next state depends on the previous state, the functional form should be used.

16. What’s the difference between controlled vs uncontrolled inputs in tricky cases?
The main difference between controlled and uncontrolled inputs is who controls the input’s value.
A controlled input is fully managed by React state. The input value always comes from state, and every change goes through React.
const [value, setValue] = useState("");
<input
  value={value}
  onChange={e => setValue(e.target.value)}
/>
This gives you full control, which is useful for validation, conditional logic, and syncing input values with other parts of the UI. However, controlled inputs can feel tricky in cases like large forms or frequent updates because every keystroke causes a re-render.
An uncontrolled input lets the browser manage the value internally. React only reads the value when needed, usually using a ref.
const inputRef = useRef(null);
<input ref={inputRef} />
This approach is simpler and can be more performant for basic forms, but it becomes harder to validate or react to changes in real time.
Tricky cases often appear when you mix both approaches.
<input value={value} defaultValue="hello" />
This causes confusion because the input starts uncontrolled and then becomes controlled, which React warns against. Inputs should be either controlled or uncontrolled for their entire lifecycle.
In short:
Use controlled inputs when React needs to manage and validate the value
Use uncontrolled inputs for simple, one-time data collection
Avoid switching between the two, as it leads to bugs and warnings
