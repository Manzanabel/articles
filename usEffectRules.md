# Stop Misusing `useEffect`: When to Use It and When Not To

React's `useEffect` is one of the most misunderstood and misused hooks, and it's easy to see why. It feels like the go-to tool whenever something needs to "happen" in your component. But that mindset often leads to **performance issues**, **infinite loops**, and unnecessary complexity.

Let’s fix that. Here's a practical guide to **when you should use `useEffect` and when you absolutely shouldn't**.

---

## ✅ The Real Purpose of `useEffect`

You should use `useEffect` only for **side effects**: actions that affect or depend on the *outside world* beyond React's render cycle.

### In simpler terms:
> 👉 Use `useEffect` when you need to interact with **browser APIs** or **external systems**.

### Common valid use cases:

| Use Case | Why it's valid |
|----------|----------------|
| `fetch()` data from an API | Triggers a network request 🌐 |
| Add event listeners (`scroll`, `resize`) | Browser events 🖱️ |
| Use `setTimeout` or `setInterval` | Timer APIs ⏰ |
| Read/write from `localStorage` | Browser storage access 💾 |
| Update `document.title` | Manipulating the DOM 📄 |
| WebSocket subscriptions | External systems 🌍 |
| Measure DOM elements (via refs) | Requires access after render 📏 |

```tsx
useEffect(() => {
  const handleScroll = () => console.log("Scrolled");
  window.addEventListener("scroll", handleScroll);

  return () => window.removeEventListener("scroll", handleScroll);
}, []);
```


## ❌ When NOT to use useEffect

Many developers use useEffect for logic that doesn't require it. Here's a quick rule of thumb:

    ❗ If something can be done during render, don’t use useEffect.

Examples of bad usage:
🔸 Deriving state from other state

// ❌ Don't do this
useEffect(() => {
  setTotal(price * quantity);
}, [price, quantity]);

// ✅ Do this instead
const total = price * quantity;

🔸 Computing flags from data

// ❌ Don't do this
```tsx
useEffect(() => {
  setIsEmpty(items.length === 0);
}, [items]);
```

// ✅ Do this instead
```tsx
const isEmpty = items.length === 0;
```

🔸 Triggering effects that re-run on every render due to unstable dependencies

 ❌ Can cause infinite loops
 ```tsx
useEffect(() => {
  setUserData(data);
}, [data]); // data is a new object each render
```

## 🧠 Summary: When to Use useEffect
| Use Case	| useEffect?|
|----------|------------|
| Calling an API |	✅ Yes |
| Setting a timer	| ✅ Yes |
| Working with localStorage or document.title	|✅ Yes |
| Listening to window resize or scroll	| ✅ Yes | 
| Deriving values from state/props |	❌ No|
| Conditional rendering or flags |	❌ No|
| Recomputing values that can be done inline	 | ❌ No |


## ✨ Rule of Thumb

    React renders are pure. useEffect is for escaping React’s world.
    Use it only when you have to reach outside — to the browser, the network, or time-based behavior.

Happy coding! Stop overusing useEffect and make your components cleaner, faster, and easier to reason about.
