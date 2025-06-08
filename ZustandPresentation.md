# React State Management with Zustand 🐻 

![bear](https://github.com/Manzanabel/zustand-presentation/assets/128307128/ea2f8d4f-43fa-4fa3-a8f6-f76b0839a2e0)
*Zustand's logo and "pet" is a cute bear who plays guitar.*

State management is a huge part when thinking about the structure of our React application. It can be crucial in many aspects, especially in terms of performance, but also readability. As you already know, local state along with props is not always enough: large applications very often need to access the application state from different - distant and unrelated - parts of the application. That's why new global state management tools have emerged. For example, some members of the React team came out with Redux (a third-party library for managing immutable state) and later with a built-in feature: the Context API. Redux was robust, but very difficult and verbose to approach. Based on the Flux architectural pattern, it intimidated developers of all levels. The arrival of the Context API was a relief to many React developers, who saw the workflow simplified. However, performance and readability [could still be improved](https://leewarrick.com/blog/the-problem-with-context), and Zustand managed to solve these problems(among others!).

## What is Zustand?
As you may know by now, Zustand is a state management tool that specializes in providing a fairly straightforward way to manage global state. It works just as Redux but it is way less verbose, making it intuitive and easy to set up right from the start. 

As per their website,

> “[...] lots of time was spent to deal with common pitfalls, like the dreaded zombie child problem, React concurrency, and context loss between mixed renderers. It may be the one state manager in the React space that gets all of these right.”

So it doesn’t only makes the state manager smoother but it also fixes some (rare) problems that occurred in other libraries.

## How do you use it?
As every javascript library, you first need to install it into your project using npm (or any other package manager).

`npm install zustand` 

Once in your app, in a `store` file preferably, you initialize the state as follows: consider a flight tracking application

```typescript
import { create } from 'zustand'

export const useStore = create<CurrentFlightState>()(set: SetState<Store>, get: GetState<Store>) => ({
  counter: 0
  currentFlight: null,
  currentCompany: null,
  setCurrentFlight: (currentFlight) =>
    set((state) => ({ ...state, currentFlight: currentFlight })),
  setCurrentCompany: (currentCompany) =>
    set((state) => ({ ...state, currentCompany: currentCompany })),
  setIncreaseCounter: () => get().counter + 1
}));
```

In a few lines of code, we have initialized the state and created the related actions. The `get` function provided by Zustand lets us access the `counter` variable from the store, and the `set` function modifies it.

Since the state store is a hook, you can use it anywhere. Contrary to Redux or the Context API, no context provider is required. Simply select your state, and the component will re-render when the state changes. Here is an example of how to use it in a React component:

```jsx
import { useStore } from "../store/store";

export const FlightIata= () => {
  const { currentFlight } = useStore((state) => state);

  return (
    <div className="basemap-container">
      // [...]
      <p>{currentFlight.flight.iata}</p>
    </div>
  );
};

```

However, don't be fooled by its simplicity. Zustand is also capable of managing more complex tasks. A useful one is the possibility to split the store in several slices, and eventually add a middleware to bind them if needed. Here is an example of how to create a middleware to bind several states into one:

```typescript
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

const useFlightSlice = create(() => ({
  flight: null,
}))

const usePlanesSlice = create(() => ({
  planes: []
})


export const useBoundStore = create(
  persist(
    (...a) => ({
      ...createFlightSlice(...a),
      ...createPlanesSlice(...a),
    }),
    { name: 'bound-store' },
  ),
)
``` 

In conclusion, Zustand emerges as a powerful yet user-friendly solution for managing global state in React applications. Its simplicity in setup and usage, combined with its ability to handle complex tasks and address common pitfalls, makes it a solid choice for developers seeking efficient state management. With Zustand, the process of managing state becomes smoother and more intuitive, allowing developers to focus on building clean applications without the burden of verbose code or potential performance bottlenecks. Zustand is a promising solution that's definitely worth considering.
