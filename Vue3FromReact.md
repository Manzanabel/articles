# Vue 3 for React developers: a practical memo

If you have been writing React for a few years and you are picking up Vue 3 for the first time (or coming back to it after a long break), this is the reference I wish I had. No framework war, no opinions about which is better. Just a honest map from what you already know to what Vue does instead.

## The mental model shift

React and Vue solve the same problem: building reactive UIs from components, but they have a different contract with the developer.

React is explicit and immutable. You never mutate state directly. You call a setter, React schedules a re-render, and the new value flows down. Everything is intentional and traceable.

Vue is [proxy-based](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) and mutation-friendly. When you declare reactive state, Vue wraps it in a Proxy that intercepts every read and write. This means you can mutate state directly and Vue will detect it and update the DOM. It feels almost too easy at first, especially if you have spent years avoiding mutation. Let me be clear here: I only thought proxies were a network thing, not a JS concept as well. I learned this concept in the process of understanding why destructuring broke my store... :) 

Neither approach is wrong. They are different contracts. Once you understand which contract Vue is using, everything else makes sense.

## Function by function comparison

This is the core of the memo. If you know what you want to do in React, here is how you do it in Vue 3.

**Declaring reactive state**

In React you use `useState`. In Vue you use `ref()` for primitives and simple values, and `reactive()` for objects. In practice, most Vue developers use `ref()` for everything, because it is more predictable and avoids a common pitfall with `reactive()` (more on that below).

```ts
// React
const [count, setCount] = useState(0)

// Vue 3
const count = ref(0)
```

One important detail: inside the `<script setup>`, you always access a `ref` with `.value`. Inside the template, Vue unwraps it automatically so you just write `{{ count }}`.

```ts
// in the script
count.value++

// in the template
{{ count }} // no .value needed
```

**Derived values**

In React you use `useMemo`. In Vue you use `computed()`. Both cache the result and only recalculate when their dependencies change.

```ts
// React
const fullName = useMemo(() => `${first} ${last}`, [first, last])

// Vue 3
const fullName = computed(() => `${first.value} ${last.value}`)
```

**Side effects**

In React you use `useEffect`. In Vue you have two options: `watch` and `watchEffect`.

`watch` is the closest equivalent to `useEffect` with a dependency array. You declare explicitly what to watch, and the callback runs when that value changes. It does not run on mount by default.

```ts
// React
useEffect(() => {
  console.log('count changed', count)
}, [count])

// Vue 3
watch(count, (newValue) => {
  console.log('count changed', newValue)
})
```

`watchEffect` is different. It runs immediately on mount, and it tracks its own dependencies automatically (no array needed). Vue figures out what you are reading inside the callback and watches it for you.

```ts
// runs on mount AND whenever count changes
watchEffect(() => {
  console.log(count.value)
})
```

**Reusable logic**

In React you extract logic into custom hooks. In Vue you extract it into composables. The concept is identical: a function that uses the reactivity system and can be shared across components. The naming convention is also the same: `use` + something.

```ts
// React hook
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth)
  useEffect(() => {
    const handler = () => setWidth(window.innerWidth)
    window.addEventListener('resize', handler)
    return () => window.removeEventListener('resize', handler)
  }, [])
  return width
}

// Vue composable
function useWindowWidth() {
  const width = ref(window.innerWidth)
  const handler = () => (width.value = window.innerWidth)
  window.addEventListener('resize', handler)
  onUnmounted(() => window.removeEventListener('resize', handler))
  return { width }
}
```

**The reactive() pitfall**

If you use `reactive()` for an object, never destructure it. Destructuring breaks the reactivity because you are copying the value out of the Proxy.

```ts
const state = reactive({ count: 0 })
const { count } = state // count is now a plain number, not reactive
```

This is why most developers default to `ref()` for everything. A `ref` wrapping an object does not have this problem because the `.value` reference stays intact.

## Two-way binding, where Vue shines

This is the area where Vue is genuinely simpler than React. In React, binding an input requires two things: a value prop and an onChange handler.

```tsx
// With React you always write both
<input value={title} onChange={(e) => setTitle(e.target.value)} />
```

In Vue, `v-model` handles both directions at once. It reads the value and updates it on input.

```vue
<!-- Vue 3: one line -->
<input v-model="title" />
```

For a form with ten fields, the difference in code volume is significant. And `v-model` works on custom components too. You can build a reusable input component that accepts `v-model` from the parent, which keeps the parent template clean.

```vue
<!-- parent -->
<MyInput v-model="title" />

<!-- MyInput.vue : under the hood v-model uses modelValue + update:modelValue -->
<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>
<template>
  <input :value="modelValue" @input="$emit('update:modelValue', $event.target.value)" />
</template>
```

This pattern is clean and explicit, and it is one of the things that makes building form-heavy applications in Vue feel natural.

## Pinia: state management without the ceremony

If you know Zustand, you already know 90% of Pinia. Both are built around the same idea: a store is just a function that returns reactive state and actions. No reducers, no action types, no dispatch.

```ts
// Zustand
const useEventStore = create((set) => ({
  events: [],
  addEvent: (event) => set((state) => ({ events: [...state.events, event] })),
}))

// Pinia
export const useEventStore = defineStore('events', () => {
  const events = ref([])
  const addEvent = (event) => events.value.push(event)
  return { events, addEvent }
})
```

The main difference from Zustand is that Pinia mutates state directly (Vue's reactivity handles it), while Zustand uses immutable updates. Other than that, the DX is very similar.

One thing to watch out for: if you destructure from a Pinia store, you lose reactivity. Same reason as the `reactive()` pitfall above. Use `storeToRefs()` to destructure safely.

```ts
// wrong: events loses reactivity
const { events } = useEventStore()

// correct
const { events } = storeToRefs(useEventStore())

// also correct: access through the store object
const store = useEventStore()
// then use store.events in the template
```

Pinia is also fully integrated with Vue DevTools, which makes debugging store state as easy as React DevTools with Redux or Zustand.

## Quick reference table

| What you want to do | React | Vue 3 |
|---|---|---|
| Reactive primitive | `useState` | `ref()` |
| Reactive object | `useState` with object | `reactive()` or `ref()` |
| Derived value | `useMemo` | `computed()` |
| Side effect on change | `useEffect` with deps | `watch()` |
| Side effect on mount + change | `useEffect` without deps | `watchEffect()` |
| Reusable logic | custom hook | composable |
| Two-way input binding | value + onChange | `v-model` |
| Global state | Redux / Zustand / Context | Pinia |
| Routing | React Router | Vue Router |

## Final thought

The transition from React to Vue 3 is smaller than it looks. The concepts are the same: reactivity, derived state, side effects, reusable logic, global store. The syntax and the contract are different, but the mental model transfers almost directly.

The thing that takes the most getting used to is trusting Vue's mutation model. After years of immutability in React, writing `myArray.value.push(item)` and watching the UI update feels strange. But it works, and once you stop fighting it, it is genuinely pleasant to write.
